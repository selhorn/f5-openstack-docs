.. _openstack-lbaas-home:

F5 OpenStack LBaaS Integration
==============================

The |oslbaas| orchestrates BIG-IP Application Delivery Controllers (ADCs) with OpenStack Networking (Neutron) services.
The Integration consists of the `F5 OpenStack BIG-IP Controller`_ and `F5 OpenStack LBaaSv2 Driver`_, which work together to configure F5 BIG-IP Local Traffic Manager (LTM) objects via the `OpenStack Networking API`_.

.. _os-lbaas-prereqs:

General Prerequisites
---------------------

The |oslbaas| documentation set assumes that you:

- already have an operational |os-deployment| with |neutron| and |heat| installed; [#partners]_
- are familiar with `OpenStack Horizon`_ and the `OpenStack CLI`_; and
- are familiar with BIG-IP LTM concepts, the BIG-IP configuration utility, and ``tmsh`` commands.

.. seealso::

   See the :ref:`F5 OpenStack Solution Test Plan <agent:solution-test-plan>` for information about minimum supported deployments.


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

Key OpenStack Concepts
----------------------

The |oslbaas| integration provides under-the-cloud multi-tenant infrastructure L4-L7 services for Neutron tenants.
In addition to community OpenStack participation, F5 maintains :ref:`partnerships <f5ospartners>`_ with several OpenStack platform vendors.


Community OpenStack and platform vendor tests exercise the generic LBaaSv2 integration. F5 OpenStack tests exercise F5-specific capabilities across multiple network topologies. They are complementary to community and platform vendor tests.

All F5 OpenStack tests are available in the same open source repository as the product code. They may be executed via tempest and tox, consistent with the OpenStack community, to allow self-validation of a deployment.

Use cases are based on real-world scenarios that represent repeatable deployments of the most common features used in F5 OpenStack integrations. Use case tests validate the combination of OpenStack, F5 BIG-IP ADC and F5 OpenStack products.


.. include:: /reuse/see-also-lbaas.rst



.. rubric:: Footnotes
.. [#partners] Unsure how to get started with OpenStack? Consult one of F5's :ref:`OpenStack Platform Partners <f5ospartners>`.

.. _OpenStack Networking API: https://developer.openstack.org/api-ref/networking/v2/
