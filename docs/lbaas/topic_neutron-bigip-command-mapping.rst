.. _neutron-bigip-command-mapping:

Neutron to BIG-IP Command Mapping
=================================

When you issue :code:`neutron lbaas` commands on your OpenStack Neutron controller, the |agent-long| configures objects on your BIG-IP device(s).
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


The sections below cover the settings |agent| applies to a standalone, :term:`overcloud` BIG-IP device.
The actual settings applied for a given command can vary depending on your existing BIG-IP device configurations and network architecture.


Start the |agent-long|
----------------------

.. rubric:: :command:`systemctl start f5-openstack agent`
1. |agent| reads the :code:`vtep` `self IP`_ defined in the |agent| config file.
2. BIG-IP advertises the :code:`vtep` IP address.
3. |agent| advertises the :code:`vtep` self IP address to Neutron as its
   ``tunneling_ip``.
4. |driver| adds a new port for the :code:`vtep` to the OVS switch.
5. |agent| adds profiles for all tunnel types to the BIG-IP device.

Create a Neutron LBaaS Load Balancer
------------------------------------

.. rubric:: :command:`neutron lbaas-loadbalancer-create`
1. |agent| creates a new BIG-IP partition.
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

Create a Neutron LBaaS Listener
-------------------------------

.. rubric:: :command:`neutron lbaas-listener-create`
|agent| creates a new BIG-IP virtual server in the specified partition using the `Fast L4`_ protocol.

The virtual server uses the IP address Neutron assigned to the load balancer and
the route domain created for the load balancer.
If using tunnels, only the tunnel assigned to the load balancer handles traffic.

For secure listeners using the :code:`TERMINATED_HTTPS` protocol: [#tablefn6]_

- |agent| fetches the certificate/key container from Barbican.
- |agent| adds the key and certificate to the BIG-IP device(s).
- |agent| creates a custom SSL profile using ``clientssl`` as the parent profile.
- |agent| adds the new SSL profile to the virtual server.

Create a Neutron LBaaS Pool
---------------------------

.. rubric:: :command:`neutron lbaas-pool-create`
|agent| adds a new pool to the specified virtual server.


Create a Neutron LBaaS Member
-----------------------------

.. rubric:: :command:`neutron lbaas-member-create`
- |agent| adds a new member to the specified pool using the IP address and port
  defined in the command.
If there is a Neutron port associated with the specified IP address and subnet, the |agent-long| creates a forwarding database (FDB) entry for the member on the BIG-IP
  device(s). [#tablefn7]_

**Notes:**

- When you add a member to a pool for the first time, the BIG-IP pool status
changes.
- When you create a member with a specific IP address for the first time, the |agent-long| also creates a new `BIG-IP node`_.

Create a Neutron LBaaS Health Monitor
-------------------------------------

.. rubric:: :command:`neutron lbaas-healthmonitor-create`

|agent| creates a new BIG-IP health monitor for the specified pool.

- If the health monitor is the first created for the specified pool, the
  pool status shown on the BIG-IP will change.
- Health monitors directly affect the status and availability of pools and
  members on the BIG-IP.
  Any additions or changes may result in a status change for the specified pool.




.. rubric:: Footnotes:
.. [#tablefn4] If using :ref:`global routed mode`, |agent| doesn't create a tunnel. Instead, all traffic goes to the load balancer's self IP address.
.. [#tablefn5] Found in the :ref:`L3 Segmentation Mode Settings` section.
.. [#tablefn6] See :ref:`Certificate Manager/SSL Offloading`.
.. [#tablefn7] The |agent| will not create a FDB entry if the pool member IP address and subnet don't have a corresponding Neutron port. Warnings print to the :code:`f5-openstack-agent` and :code:`neutron-server` logs if the pool member doesn't have a corresponding Neutron port.

.. _self IP: https://support.f5.com/kb/en-us/products/big-ip_ltm/manuals/product/tmos-routing-administration-12-0-0/6.html#conceptid
.. _SNAT automap: https://support.f5.com/kb/en-us/products/big-ip_ltm/manuals/product/tmos-routing-administration-12-0-0/8.html#unique_375712497
.. _Fast L4: https://support.f5.com/kb/en-us/products/big-ip_ltm/manuals/product/ltm-profiles-reference-13-0-0/5.html
.. _BIG-IP node: https://support.f5.com/kb/en-us/products/big-ip_ltm/manuals/product/ltm-basics-13-0-0/3.html
