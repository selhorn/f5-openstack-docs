.. _neutron-bigip-command-mapping:

Neutron to BIG-IP Command Mapping
=================================

When you issue a :code:`neutron lbaas` commands on your OpenStack Neutron controller, the |agent-longconfigures objects on your BIG-IP device(s).
Here, we provide some insight into what's happening behind the scenes.


.. tip::

   To view the actual API calls the |agent-long| sends to the BIG-IP device(s), :ref:`set the F5 agent's DEBUG level <lbaas-set-log-level>` to 'True' and view the logs (:file:`/var/log/neutron/f5-openstack-agent.log`).

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

   =========================================================== =================================================================================
   Command                                                     Action(s)
   =========================================================== =================================================================================
   :command:`systemctl start f5-openstack agent`               1. |agent| reads the :code:`vtep` `self IP`_ defined in the |agent| config file.
                                                               2. BIG-IP advertises the :code:`vtep` IP address.
                                                               3. |agent| advertises the :code:`vtep` self IP address to Neutron as its
                                                                  ``tunneling_ip``.
                                                               4. |driver| adds a new port for the :code:`vtep` to the OVS switch.
                                                               5. |agent| adds profiles for all tunnel types to the BIG-IP device.
   ----------------------------------------------------------- ---------------------------------------------------------------------------------
   :command:`neutron lbaas-loadbalancer-create`                1. |agent| creates a new BIG-IP partition.
                                                               2. |agent| creates BIG-IP FDB records for all peers in the network.
                                                               3. |agent| creates a new BIG-IP route domain.
                                                               4. |agent| creates a new BIG-IP self IP on the specified subnet. This is the IP
                                                                  address at which the BIG-IP device can receive traffic for this load balancer.
                                                               5. |agent| creates a new tunnel.

                                                                  - uses the :code:`vtep` as the local address and
                                                                  - uses the vxlan profile created when the |agent| started [#tablefn4]_

                                                               6. |agent| creates a SNAT pool list/SNAT translation list on the BIG-IP device(s).

                                                                  - Set the number of SNAT addresses to create with the
                                                                  ``f5_snat_addresses_per_subnet`` setting in the :ref:`agent configuration file`.
                                                                  [#tablefn5]_

                                                               7. |driver| adds a Neutron port for each SNAT address.

                                                                  - If SNAT mode is off and SNAT addresses is set to ``0``, the BIG-IP
                                                                    acts as a gateway and handles all return traffic from members.
                                                                  - If SNAT mode is on & SNAT addresses is set to ``0``, the BIG-IP device uses
                                                                    `SNAT automap`_.
   ----------------------------------------------------------- ---------------------------------------------------------------------------------
   :command:`neutron lbaas-listener-create`                    |agent| ceates a new BIG-IP virtual server in the partition for the specified
                                                               load balancer.

                                                               - uses Fast L4 by default
                                                               - If persistence is configured, Standard is used.
                                                               - Uses the IP address assigned to the load balancer by Neutron.
                                                               - Uses the route domain created for the new partition when the load balancer was
                                                                 created.
                                                               - Traffic is restricted to the tunnel assigned to the load balancer.

                                                               If the listener ``--protocol`` is ``TERMINATED_HTTPS``: [#tablefn6]_

                                                               - |agent| fetches the certificate/key container from Barbican; add the URI to the
                                                                 ``default_tls_container_ref`` config parameter to tell |agent| where to find it.
                                                               - |agent| adds the key and certificate the BIG-IP device(s).
                                                               - |agent| creates a custom SSL profile (uses ``clientssl`` as the parent profile).
                                                               - |agent| adds the new SSL profile to the virtual server.
   ----------------------------------------------------------- ---------------------------------------------------------------------------------
   :command:`neutron lbaas-pool-create`                        |agent| adds a new pool to the specified virtual server.
   ----------------------------------------------------------- ---------------------------------------------------------------------------------
   :command:`neutron lbaas-member-create`                      |agent| adds a new member to the specified pool using the IP address and port
                                                               supplied in the command.

                                                               - If the member is the first created for the specified pool, the pool status
                                                                 displayed on the BIG-IP device(s) will change.
                                                               - If the member is the first created with the supplied IP address, |agent| also
                                                                 creates a new node.
                                                               - |agent| creates a forwarding database (FDB) entry for the member on the BIG-IP
                                                                 device(s). [#tablefn7]_
   ----------------------------------------------------------- ---------------------------------------------------------------------------------
   :command:`neutron lbaas-healthmonitor-create`               |agent| creates a new BIG-IP health monitor for the specified pool.

                                                               - If the health monitor is the first created for the specified pool, the
                                                                 pool status shown on the BIG-IP will change.
                                                               - Health monitors directly affect the status and availability of pools and
                                                                 members on the BIG-IP.
                                                                 Any additions or changes may result in a status change for the specified pool.
   =========================================================== =================================================================================



.. rubric:: Footnotes:
.. [#tablefn4] If using :ref:`global routed mode`, |agent| doesn't create a tunnel. Instead, all traffic goes to the load balancer's self IP address.
.. [#tablefn5] Found in the :ref:`L3 Segmentation Mode Settings` section.
.. [#tablefn6] See :ref:`Certificate Manager/SSL Offloading`.
.. [#tablefn7] The |agent| will not create a FDB entry if the pool member IP address and subnet don't have a corresponding Neutron port. Warnings print to the :code:`f5-openstack-agent` and :code:`neutron-server` logs if the pool member doesn't have a corresponding Neutron port.

.. _self IP: https://support.f5.com/kb/en-us/products/big-ip_ltm/manuals/product/tmos-routing-administration-12-0-0/6.html#conceptid
.. _SNAT automap: https://support.f5.com/kb/en-us/products/big-ip_ltm/manuals/product/tmos-routing-administration-12-0-0/8.html#unique_375712497
