.. _lbaas-manage-vcmp-clusters:

Manage BIG-IP vCMP Clusters
===========================

Overview
--------

Virtual Clustered Multiprocessing™ (vCMP) is a feature of the BIG-IP system that allows you to run multiple instances of BIG-IP software on a single hardware platform.
vCMP allocates a specific share of the hardware resources to each BIG-IP instance, or :term:`vCMP guest`.

A vCMP guest consists of a TMOS instance and one or more BIG-IP modules.
The :term:`vCMP host` allocates a share of the hardware resources to each guest; each guest also has its own management IP address, self IP addresses, virtual servers, and so on.
In this way, each guest can effectively receive and process application traffic with no knowledge of other guests on the system.

F5 LBaaSv2 allows you to manage vCMP guests in the same way that you would a physical BIG-IP device or BIG-IP Virtual Edition.
See the |agent-long| :ref:`configuration instructions <agent:configuration>` for details.

Use Case
--------

When used with vCMP in a flat network or VLAN, the |agent-long| can manage one or more vCMP hosts, each of which can have one or more guests.
Guests on the same, or different, vCMP hosts can be configured to operate as a :term:`device service cluster`.
If a vCMP host fails (taking its guests with it), another vCMP host with guests configured as part of the cluster can take over managing its traffic.
This provides a high degree of redundancy, while requiring fewer physical resources.
vCMP also allows you to delegate management of the BIG-IP software in each instance to individual administrators.
This means users who need to manage LBaaS objects don't need to be given full administrative access to the BIG-IP, only to the host & guests allotted for their project(s).

Prerequisites
-------------

- Licensed, operational BIG-IP chassis with support for vCMP.
- Licensed, operational BIG-IP vCMP guest running on a vCMP host.
- Administrative access to the vCMP host(s) and guest(s) you will manage with F5 LBaaSv2.
- F5 :ref:`agent <agent:home>` and :ref:`service provider driver <Install the F5 LBaaSv2 Driver>` installed on the Neutron controller.
- Knowledge of `BIG-IP vCMP <https://support.f5.com/kb/en-us/products/big-ip_ltm/manuals/product/vcmp-administration-appliances-12-1-1/1.html>`_ configuration and administration.

Caveats
-------
.. todo:: check whether the below is true:

- VLAN and FLAT are the only supported network types that can be used with vCMP.
  You can not use vCMP with VxLAN or GRE tunnel configurations.

Configuration
-------------

Edit the :ref:`Device Driver/iControl driver settings <agent:driver-settings>` and :ref:`L2 Segmentation Mode <agent:l2-segmentation-settings>` sections of the |agent-long| :ref:`configuration file <agent:configuration-file`.

#. Add the ``icontrol_vcmp_hostname``. Multiple values can be comma-separated.

.. code-block:: text
   :emphasize-lines: 8

   #
   icontrol_vcmp_hostname = 1.2.3.4
   #

#. Configure the ``icontrol_hostname`` parameter with the IP address(es) of the vCMP guest(s):

.. code-block:: text
   :emphasize-lines: 19

   #
   icontrol_hostname = 10.11.12.13, 14.15.16.17
   #

#. Set ``advertised_tunnel_types`` to ``vlan`` or ``flat``, as appropriate for your environment.

   .. important::

      If the ``advertised_tunnel_types`` setting in the Agent Configuration File is left empty, as in the example below, the ML2 plugin ``provider:network_type`` should be set to FLAT or VLAN.

   .. code-block:: text
      :emphasize-lines: 10

      #
      advertised_tunnel_types =
      #


.. seealso::

   * See the `BIG-IP vCMP documentation`_ for more information about vCMP.

.. _BIG-IP vCMP documentation: https://support.f5.com/kb/en-us/products/big-ip_ltm/manuals/product/vcmp-administration-appliances-12-1-1/1.html?sr=57167911

