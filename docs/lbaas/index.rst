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


|driver-long|
-------------

The |driver-link|, or |driver|, is F5's OpenStack Neutron service provider driver.
It picks up Neutron LBaaS calls from the RPC messaging queue and assigns them to the |agent-long|.

.. figure:: /_static/media/f5-lbaas-architecture.png
   :align: center
   :scale: 100%
   :alt: Diagram showing the architecture of the F5 OpenStack LBaaS integration. A user issues a neutron lbaas command; the F5 LBaaSv2 driver assigns the task from the Neutron RPC messaging queue to the F5 OpenStack BIG-IP Controller. The BIG-IP Controller periodically reports its status to the Neutron database.

   F5 OpenStack LBaaS Integration Architecture


|agent-long|
------------

The |agent-link|, or |agent|, translates from "OpenStack" to "F5".
It receives tasks from the Neutron RPC messaging queue, converts them to `iControl REST`_ API calls (using the `F5 Python SDK`_), and sends the calls to the BIG-IP device(s).

.. figure:: /_static/media/f5-lbaas-agent-to-BIG-IP.png
   :align: center
   :scale: 100%
   :alt: Diagram showing the operation of the F5 OpenStack BIG-IP Controller. A user issues a neutron lbaas command; the F5 LBaaSv2 driver assigns the task to the F5 OpenStack BIG-IP Controller; the BIG-IP Controller sends the command to the BIG-IP device as an iControl REST API call to add or edit the requested object.

   F5 OpenStack BIG-IP Controller traffic flow



.. include:: /reuse/see-also-lbaas.rst



.. rubric:: Footnotes
.. [#partners] Unsure how to get started with OpenStack? Consult one of F5's :ref:`OpenStack Platform Partners <f5ospartners>`.

.. _OpenStack Networking API: https://developer.openstack.org/api-ref/networking/v2/
