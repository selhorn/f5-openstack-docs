.. _lbaas-agent-redundancy:

Redundancy and Scale Out
========================

.. note::

   The term "host" can mean a lot of things. It can be an OpenStack compute node, the Neutron controller, a virtual machine, a container, etc.
   The important takeaway is that the :code:`hostname` for each |agent| instance must be unique.

Multiple |agent| instances can manage a single BIG-IP device or :term:`cluster` if **each instance runs on a separate host**.
Running |agent| instances on different hosts helps ensure that the |oslbaas| processes remain alive and functional if a host goes down.
Spreading the request load for an environment across multiple |agent| instances helps to avoid |agent| overload and loss of functionality.

.. warning::

   **F5 Networks does not currently support container service deployments in OpenStack.**

If you are well versed in containerized environments, you can run each |agent| instance in a separate container on your Neutron controller.
If the service provider driver is in the container's build context, you don't need to install it in each container.

- The :file:`neutron.conf` and :file:`neutron-lbaas.conf` files must be present in each container.
- The service provider driver **does not** need to run in the container if you're building from the Neutron controller.

Prerequisites
-------------

- Administrator access to both the BIG-IP device(s) and the OpenStack cloud.
- All hosts running the |oslbaas| must have the Neutron and Neutron LBaaS packages installed.
- All hosts running the |oslbaas| must use the same Neutron database.

Caveats
-------

- You **can not run multiple** |agent| **instances on the same host** if you want them to manage the same BIG-IP device or :term:`cluster`.
- In the standard multi-agent deployment, you can't specify which |agent| instance you want to use to create a new load balancer (meaning you can't choose which BIG-IP device/cluster to create the new partition on).
- Use :ref:`differentiated service environments <lbaas-differentiated-service-env>` if you need a greater degree of control over which |agent| instance(s) handle specific LBaaS requests.

Configuration
-------------

#. Copy the Neutron and Neutron LBaaS configuration files from the Neutron controller to each host on which you want to run an |agent| instance.

   .. code-block:: console

      cp /etc/neutron/neutron.conf <hostname>:/etc/neutron/neutron.conf
      cp /etc/neutron/neutron_lbaas.conf <hostname>:/etc/neutron/neutron_lbaas.conf

#. :ref:`Install the F5 OpenStack BIG-IP Controller <agent:install>` on each host.

#. Copy your |agent-long| configuration file from the Neutron controller to each host.

   .. code-block:: console

      cp /etc/neutron/services/f5/f5-openstack-agent.ini <hostname>:/etc/neutron/services/f5/f5-openstack-agent.ini

   .. tip::

      * If you are managing an :term:`active-standby pair` or :term:`cluster` with `config sync`_ turned on:

        - Set the :code:`ha_type` to :code:`standalone`.
        - Provide the iControl endpoint for one (1) of the BIG-IP devices in the cluster.

      * If you are managing a :term:`cluster` that has `config sync`_ turned on for a :term:`device service group` within the cluster:

        - Set the :code:`ha_type` to :term:`pair` or :term:`scalen`.
        - Provide the iControl endpoint for one (1) of the BIG-IP devices in the device service group and the endpoint for a device outside the group (:code:`pair`).

          --OR--

        - Provide the iControl endpoint for one (1) of the BIG-IP devices in the device service group and the endpoint for each device in the cluster that is not automatically syncing its configurations with the group. (:code:`scalen`)


#. Start the |agent-long| on each host.

   .. include:: /_static/reuse/start-f5-agent.rst


.. seealso::

   * :ref:`Configure the F5 OpenStack Agent`
   * :ref:`Manage BIG-IP Clusters with F5 LBaaSv2`
   * :ref:`Manage Multi-Tenant BIG-IP Devices with F5 LBaaSv2`
   * :ref:`Differentiated Service Environments`


.. _config sync: https://support.f5.com/kb/en-us/products/big-ip_ltm/manuals/product/bigip-system-device-service-clustering-administration-13-0-0/5.html
