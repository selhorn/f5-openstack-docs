.. _lbaas-manage-vcmp-clusters:

.. _lbaas-manage-vcmp-systems:

Manage BIG-IP vCMP Systems
==========================

Overview
--------

You can use `Virtual Clustered Multiprocessing`_ (vCMP) to run multiple instances of BIG-IP software on a single hardware platform.
The :term:`vCMP host` allocates a share of the hardware resources to each :term:`vCMP guest`.
Each guest has its own management IP address, self IP addresses, virtual servers, and so on.
Each guest can receive and process application traffic with no knowledge of other guests on the system.
vCMP  allows you to delegate management of BIG-IP instances, meaning users who need to manage LBaaS objects don't need to have full administrative access to the BIG-IP device.

Use Case
--------

The |agent-long| can manage one (1) or more vCMP hosts with one (1) or more guests per host.
You can configure vCMP hosts as a :term:`device service cluster`. [#vcmpcluster]_
If a vCMP host fails (taking its guests with it), another vCMP host in the cluster can take over managing its traffic.

Prerequisites
-------------

- Licensed, operational `BIG-IP chassis`_ with support for vCMP.
- Licensed, operational BIG-IP vCMP guest running on a vCMP host.
- Administrative access to the vCMP host(s) and guest(s) you will manage with F5 LBaaSv2.
- F5 :ref:`agent <agent:home>` and :ref:`service provider driver <Install the F5 LBaaSv2 Driver>` installed on the Neutron controller.
- Knowledge of `BIG-IP vCMP <https://support.f5.com/kb/en-us/products/big-ip_ltm/manuals/product/vcmp-administration-appliances-12-1-1/1.html>`_ configuration and administration.

Caveats
-------
.. todo:: check whether the below is still true.

- VLAN and FLAT are the only network types supported for use with vCMP.

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

   #
   icontrol_hostname = 10.11.12.13, 14.15.16.17
   #

#. Set ``advertised_tunnel_types`` to ``vlan`` or ``flat``, as appropriate for your environment.

   .. tip`::

      You can leave the  ``advertised_tunnel_types`` setting empty, as in the example below, if the ML2 plugin ``provider:network_type`` is FLAT or VLAN.

   .. code-block:: text
      :emphasize-lines: 10

      #
      advertised_tunnel_types =
      #

.. rubric:: Footnotes
.. [#vcmpcluster] See `Device Service Clustering for vCMP Systems <https://support.f5.com/kb/en-us/products/big-ip_ltm/manuals/product/vcmp-administration-appliances-12-1-1/4.html>`_


.. _Virtual Clustered Microprocessing: https://support.f5.com/kb/en-us/products/big-ip_ltm/manuals/product/vcmp-administration-appliances-12-1-1/1.html
.. _BIG-IP chassis: https://f5.com/products/deployment-methods/hardware
