.. _f5-openstack-lbaasv2-coding-example:

.. _lbaas-basic-loadbalancer:

How to Configure a Basic Loadbalancer
=====================================

The series of code samples provided here demonstrate how to configure a basic loadbalancer via the OpenStack Neutron CLI with the |oslbaas|.
The `OpenStack CLI`_ documentation has a full list of all :code:`neutron lbaas` commands.

.. important::

   The LBaaSv2 CLI commands begin with :code:`lbaas-`.
   Commands beginning with :code:`lb-` are part of the deprecated LBaaSv1 project.

.. todo:: add neutron cli response text and `show` info after each command

Create a load balancer
----------------------

.. note::

   Neutron LBaaS :code:`loadbalancer` == BIG-IP :term:`partition`

.. todo:: add note about naming convention applied when you create a new load balancer

Specify the name you want to assign to the load balancer and the existing OpenStack subnet you want to assign to it.

.. code-block:: shell

   $ neutron lbaas-loadbalancer-create --name lb1 private-subnet


Create a listener
-----------------

.. note::

   Neutron LBaaS listener == BIG-IP virtual server

Specify the name you want to assign to the virtual server; the name of the load balancer (BIG-IP partition) you want to create the virtual server for; and the protocol type and port you'd like to use.

.. code-block:: shell

   $ neutron lbaas-listener-create --name vs1 --loadbalancer lb1 --protocol HTTP --protocol-port 8080


Create a pool
-------------

When you create a pool, specify the name you want to assign to the pool; the :ref:`load balancing method <tbd>` you want to use; the name of the listener (virtual server) you want to attach the pool to; and the protocol type the pool should use.

.. code-block:: shell

   $ neutron lbaas-pool-create --name pool1 --lb-algorithm ROUND_ROBIN --listener vs1 --protocol HTTP


Create a pool member
--------------------

When creating a pool member, specify the existing OpenStack subnet you want to assign to it; the IP address the member should process traffic on; the protocol port; and the name or UUID of the pool you want to attach the member to.

.. code-block:: shell

   $ neutron lbaas-member-create --subnet private-subnet --address 172.16.101.89 --protocol-port 80 pool1


Create a health monitor
-----------------------

When creating a health monitor, specify the delay; monitor type; number of retries; timeout period; and the name of the pool you want to monitor.

.. code-block:: shell

   $ neutron lbaas-healthmonitor-create --delay 3 --type HTTP --max-retries 3 --timeout 3 --pool pool1


What's Next
-----------

Verify that all of your Neutron LBaaS objects were added to the BIG-IP device using the BIG-IP configuration utility.

#. Log in to the BIG-IP configuration utility at the management IP address (e.g., :code:`https://1.2.3.4/tmui/login.jsp`).
#. Use the :guilabel:`Partition` drop-down menu to select the correct partition for your load balancer.
#. Go to :menuselection:`Local traffic --> Virtual Servers` to view your new listener, pool, pool member, and health monitor.


