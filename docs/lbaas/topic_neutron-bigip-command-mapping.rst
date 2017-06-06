.. _neutron-bigip-command-mapping:

Neutron to BIG-IP Command Mapping
=================================

When you issue a :code:`neutron lbaas` commands on your OpenStack Neutron controller, the |agent-longconfigures objects on your BIG-IP device(s).
Here, we provide some insight into what's happening behind the scenes.


.. tip::

   To view the actual API calls the |agent-long| sends to the BIG-IP device(s), :ref:`set the F5 agent's DEBUG level <>` to 'True' and view the logs (:file:`/var/log/neutron/f5-openstack-agent.log`).

F5 LBaaSv2 uses the `f5-sdk <http://f5-sdk.readthedocs.io/en/latest/>`_ to communicate with BIG-IP via the iControl® REST API. The table below shows the corresponding iControl endpoint and BIG-IP object for each neutron lbaas- ‘create’ command.

.. table:: Neutron Command to iControl REST API endpoint Mapping

   ==============================================  ==================================================================================================
   Neutron command                                 iControl REST API endpoint
   ==============================================  ==================================================================================================
   :command:`neutron lbaas-loadbalancer-create`    :code:`https://<icontrol_endpoint>:443/mgmt/tm/sys/folder/~Project_<os_tenant_id>`
   ----------------------------------------------  --------------------------------------------------------------------------------------------------
   :command:`neutron lbaas-listener-create`        :code:`https://<icontrol_endpoint>:443/mgmt/tm/ltm/virtual/`
   ----------------------------------------------  --------------------------------------------------------------------------------------------------
   :command:`neutron lbaas-pool-create`            :code:`https://<icontrol_endpoint>:443/mgmt/tm/ltm/pool/`
   ----------------------------------------------  --------------------------------------------------------------------------------------------------
   :command:`neutron lbaas-member-create`          :code:`https://<icontrol_endpoint>:443/mgmt/tm/ltm/pool/~Project_<os_tenant_id>~pool1/members/`
   ----------------------------------------------  --------------------------------------------------------------------------------------------------
   :command:`neutron lbaas-healthmonitor-create`   :code:`https://<icontrol_endpoint>:443/mgmt/tm/ltm/monitor/http/`
   ==============================================  ==================================================================================================


The table below provides an example of the settings |agent| applies to a standalone, :term:`overcloud` BIG-IP device.
The actual settings applied may vary depending on your existing BIG-IP device configurations and network architecture.

.. table:: Neutron command to BIG-IP configuration mapping

======================================     =================================================================================
Neutron command                            |agent| action(s) taken
======================================     =================================================================================
``systemctl start f5-openstack agent``     1. |agent| reads the vtep `self IP`_ defined in the |agent| config file.
                                           2. BIG-IP advertises the vtep's IP address.
                                           3. The self IP address is advertised to Neutron as the agent's
                                              ``tunneling_ip``.
                                           4. A new port for the vtep is added to the OVS switch.
                                           5. Profiles for all tunnel types are created on the BIG-IP. [#tablefn1]_
--------------------------------------     ---------------------------------------------------------------------------------
``neutron lbaas-loadbalancer-create``      1. A new partition is created using the prefix [#tablefn2]_ and tenant ID [#tablefn3]_.
                                           2. New fdb records are added for all peers in the network.
                                           3. A new route domain is created.
                                           4. A new self IP where the BIG-IP can receive traffic is created on the
                                              specified subnet.
                                           5. A new tunnel is created, using the vtep as the local address (uses the
                                              vxlan profile created when the |agent| was first started). [#tablefn4]_
                                           6. A SNAT pool list / SNAT translation list is created on the BIG-IP.

                                              - The number of SNAT addresses that will be created is defined in the agent
                                                config file. [#tablefn5]_

                                           7. A neutron port is created for each SNAT address.

                                              - If SNAT mode is turned off and SNAT addresses is set to ``0``, the BIG-IP
                                                will act as a gateway so return traffic from members is always routed
                                                through it.
                                              - If SNAT mode is turned on & SNAT addresses is set to ``0``, `SNAT automap`_
                                                will be used.
--------------------------------------     ---------------------------------------------------------------------------------
``neutron lbaas-listener-create``          1. A new virtual server is created in the tenant partition on the BIG-IP.

                                              - Attempts to use Fast L4 by default.
                                              - If persistence is configured, Standard is used.
                                              - Uses the IP address assigned to the load balancer by Neutron.
                                              - Uses the route domain that was created for the new partition when the
                                                load balancer was created.
                                              - Traffic is restricted to the tunnel assigned to the load balancer.

                                           If the listener ``--protocol`` is ``TERMINATED_HTTPS``: [#tablefn6]_

                                              - The certificate/key container is fetched from Barbican using the URI
                                                defined by the ``default_tls_container_ref`` config option.
                                              - The key and certificate are imported to the BIG-IP.
                                              - A custom SSL profile is created using ``clientssl`` as the parent profile.
                                              - The SSL profile is added to the virtual server.
--------------------------------------     ---------------------------------------------------------------------------------
``neutron lbaas-pool-create``              A new pool is created in the tenant partition on the BIG-IP.
                                              - It is assigned to the virtual server (or, listener) specified in the
                                              command.
--------------------------------------     ---------------------------------------------------------------------------------
``neutron lbaas-member-create``            A new member is created in the specified pool using the IP address and port
                                           supplied in the command.

                                           - If the member is the first created for the specified pool, the pool
                                             status will change on the BIG-IP.
                                           - If the member is the first created with the supplied IP address, a new
                                             node is also created.
                                           - The |agent| creates a forwarding database (FDB) entry for the member
                                             on the BIG-IP device(s). [#tablefn7]_
--------------------------------------     ---------------------------------------------------------------------------------
``neutron lbaas-healthmonitor-create``     A new health monitor is created on the BIG-IP for the specified pool.

                                           - If the health monitor is the first created for the specified pool, the
                                             pool status will change on the BIG-IP.
                                           - Health monitors directly affect the status and availability of pools and
                                             members on the BIG-IP. Any additions or changes may result in a status
                                             change for the specified pool.
======================================     =================================================================================



.. rubric:: Footnotes:
.. [#tablefn1] This is done for all tunnel types, not just those configured as the ``advertised_tunnel_types`` in the :ref:`L2 Segmentation Mode Settings`.
.. [#tablefn2] Configured in ``Environment Settings --> environment_prefix``. The default prefix is ``Project``.
.. [#tablefn3] Run ``openstack project list`` to get a list of configured tenant names and IDs.
.. [#tablefn4] If using :ref:`global routed mode`, all traffic is directed to the self IP (no tunnel is created).
.. [#tablefn5] Configured in :ref:`L3 Segmentation Mode Settings` --> ``f5_snat_addresses_per_subnet``.
.. [#tablefn6] See :ref:`Certificate Manager / SSL Offloading`.
.. [#tablefn7] The |agent| will not create a FDB entry if the pool member IP address and subnet don't have a corresponding Neutron port. Warnings print to the :code:`f5-openstack-agent` and :code:`neutron-server` logs if the pool member doesn't have a corresponding Neutron port.

.. _self IP: https://support.f5.com/kb/en-us/products/big-ip_ltm/manuals/product/tmos-routing-administration-12-0-0/6.html#conceptid
.. _SNAT automap: https://support.f5.com/kb/en-us/products/big-ip_ltm/manuals/product/tmos-routing-administration-12-0-0/8.html#unique_375712497
