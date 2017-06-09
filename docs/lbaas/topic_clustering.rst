.. _lbaas-manage-clusters:

Manage BIG-IP Clusters
======================

You can use the |oslbaas| to manage BIG-IP :term:`device service clusters`, making :term:`high availability`, :term:`mirroring`, and :term:`failover` services in your OpenStack cloud.

Clustering provides a greater degree of redundancy than a standalone device offers.
It helps to avoid service interruptions that could otherwise occur if a device should go down.

The |oslbaas| can manage BIG-IP `Sync-Failover device groups`_ when the :ref:`High Availability mode <HA mode>` is set to :term:`pair` or :term:`scalen` .

.. figure:: /_static/media/f5-lbaas-scalen-cluster.png
   :alt: BIG-IP scalen cluster diagram
   :scale: 60%

   BIG-IP scalen cluster

The |agent-long| expects to find a specific number of iControl endpoints (the ``icontrol_hostname`` :ref:`configuration parameter <agent:configuration-parameters>` based on the ``f5_ha_type``, as noted below.

.. table:: |oslbaas| high availability (HA) options

   ================= ========================================
   HA type           Number of iControl endpoints expected
   ================= ========================================
   standalone        1
   ----------------- ----------------------------------------
   pair              2
   ----------------- ----------------------------------------
   scalen            > 2
   ================= ========================================

F5 LBaaSv2 and BIG-IP Auto-sync
```````````````````````````````

.. important::

   The |agent-long| applies LBaaS configuration changes to each BIG-IP :term:`device` in a cluster at the same time, in real time.
   For this reason, **do not** use `configuration synchronization`_ (config sync) in clusters managed by the |oslbaas|.

For example, if you create a load balancer for a device group using config sync, the create command will succeed on the first device in the group and fail on the others.
The failure occurs because the requested partition was created on each device in the cluster via config sync.

If you need to sync a BIG-IP device group, do so manually **after** making changes to Neutron LBaaS objects.

.. danger::

   If you must use config sync mode, set the ``f5_ha_type`` to ``standalone`` and enter the iControl endpoint for one (1) of the BIG-IP devices in the group.

   If you choose to do so, **you must manually replace the iControl endpoint** in the |agent| :ref:`configuration file` with the iControl endpoint of another device in the group if the configured device should fail.

   While it is possible to use config sync for a device group *after* creating a new load balancer, it is not recommended. This functionality has not been tested.

Prerequisites
-------------

- Administrator access to both BIG-IP devices and OpenStack cloud.
- Licensed, operational BIG-IP :term:`device service cluster`.

  .. tip::

     If you do not already have a BIG-IP cluster deployed in your network, you can use the :ref:`F5 BIG-IP: Active-Standby Cluster <heat:active-standby-cluster>` Heat template to create an :term:`overcloud` two-device cluster.


Caveats
-------

- The |agent-long| can manage clusters of two (2) to four (4) BIG-IP devices.
  Active-standby, or "pair", mode applies to two-device clusters; scalen applies to clusters of more than two (2) devices.
- The administrator login must be the same on all BIG-IP devices in the cluster.

Configuration
-------------

Edit the :ref:`device settings <agent:device-settings>` and :ref:`Device Driver/iControl driver settings <agent:driver-settings>` sections of the |agent-long| :ref:`configuration file <agent:configuration-file`.

#. Set the :ref:`HA mode` to :term:`pair` **or** :term:`scalen`.

   .. code-block:: text
      :emphasize-lines: 10

      vi /etc/neutron/services/f5/f5-openstack-agent.ini
      ...
      # HA mode
      #
      f5_ha_type = pair    \\ 2-device cluster
      f5_ha_type = scalen  \\ 2-4 device cluster
      #
      #

#. Add the iControl endpoint (IP address) for each BIG-IP device in the cluster and the admin login credentials.
   Values must be comma-separated.

   .. code-block:: text
      :emphasize-lines: 10

      #
      icontrol_hostname = 1.2.3.4,5.6.7.8
      #
      icontrol_username = myusername
      #
      icontrol_password = mypassword
      #


.. seealso::

   * :ref:`Manage BIG-IP vCMP clusters <lbaas-manage-vcmp-clusters>`


.. _BIG-IP device service clustering: https://support.f5.com/kb/en-us/products/big-ip_ltm/manuals/product/bigip-device-service-clustering-administration-12-1-1.html
.. _Sync-Failover device groups: https://support.f5.com/kb/en-us/products/big-ip_ltm/manuals/product/bigip-device-service-clustering-administration-12-1-1/4.html
.. _configuration synchronization: https://support.f5.com/kb/en-us/products/big-ip_ltm/manuals/product/bigip-device-service-clustering-administration-12-1-1/5.html
