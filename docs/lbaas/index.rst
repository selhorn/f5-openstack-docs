.. _openstack-lbaas-home:

F5 OpenStack LBaaS Integration
==============================

The |oslbaas| consists of the `F5 OpenStack BIG-IP Controller`_ and `F5 OpenStack LBaaSv2 Driver`_.
The two work together to configure F5 BIG-IP Local Traffic Manager (LTM) objects via the `OpenStack Networking API`_.

.. _os-lbaas-prereqs:

General Prerequisites
---------------------

The F5 OpenStack LBaaS Integration's documentation set assumes that you:

- already have an operational |os-deployment| with |neutron| and |heat| installed; [#partners]_
- are familiar with `OpenStack Horizon`_ and the `OpenStack CLI`_; and
- are familiar with BIG-IP LTM concepts, the BIG-IP configuration utility, and ``tmsh`` commands.


|agent-long|
------------

The |agent-link|, or |agent|, translates from "OpenStack" to "F5".
It receives tasks from the Neutron RPC messaging queue, converts them to iControl REST API calls (using the `F5 Python SDK`_), and sends the calls to the BIG-IP devices.

|driver-long|
-------------

The |driver-link|, or |driver|, is F5's OpenStack Neutron service provider driver. It picks up Neutron LBaaS calls from the RPC messaging queue and assigns them to the |agent-long|.

.. seealso::

   - quick start
   - multiple agents different environments
   -

Architecture
------------



.. include:: /reuse/see-also-lbaas.rst



.. rubric:: Footnotes
.. [#partners] Unsure how to get started with OpenStack? Consult one of F5's :ref:`OpenStack Platform Partners <f5ospartners>`.

.. _OpenStack Networking API: https://developer.openstack.org/api-ref/networking/v2/
