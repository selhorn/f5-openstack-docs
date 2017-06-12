Hierarchical Port Binding
=========================

Overview
--------

Neutron `hierarchical port binding`_ allows you to use the |oslbaas| with software-defined networking (SDN) to dynamically configure VLANs and VLAN tags for a physical BIG-IP :term:`device` or :term:`device service cluster`.
When you tell the |agent-long| what top of rack (ToR) L3 switch and port (in other words, which network segment) the BIG-IP devices are connected to, the |agent-long| can set up the BIG-IPs to process traffic for dynamically-created networks in that segment.

What are Disconnected Services?
```````````````````````````````

:dfn:`Disconnected services` are LBaaS objects that are assigned to a Neutron network that isn't bound to physical network segment yet.
These "disconnected services" automatically connect to the intended Neutron network when the |agent-long| discovers it.
The |agent-long| polling frequency and maximum wait time allow for a degree of variation in the timing of the VLAN deployment and the request to create the LBaaS objects for it.

Use Case
````````

Use heirarchical port binding if you want your :term:`undercloud` physical BIG-IP device or cluster to control traffic for   networks dynamically created via SDN.
As noted in the OpenStack documentation, this can be useful if you need your Neutron deployment to scale beyond the 4K-VLANs-per-physical network limit. [#osvlans]_

When the |agent-long| is configured with the name of a switch and the port(s) to which BIG-IP devices are connected, the LBaaSv2 driver discovers Neutron networks in that switch's network segment.
The driver provides the segmentation IDs of VLANs in the network segment to the |agent-long|, which then dynamically creates the VLAN tags required to connect LBaaS services to the BIG-IPs.


.. figure:: /_static/media/lbaasv2_hierarchical-port-binding.png
   :alt: F5 LBaaSv2 Hierarchical Port Binding
   :scale: 60%

   F5 LBaaSv2 Hierarchical Port Binding


Prerequisites
`````````````

- Operational BIG-IP :term:`device` or :term:`device cluster` licensed for SDN services.
- Administrator access to both BIG-IP device(s) and OpenStack cloud.

Caveats
```````

- The only ML2 network type supported for use with hierarchical port binding is :code:`VLAN`.
- Each |agent| instance managing a BIG-IP :term:`device service cluster` must have the same ``f5_network_segment_physical_network`` setting. [#caveat1]_
- If multiple |agent-long| instances are managing the same :ref:`service environment <lbaas-differentiated-service-env>`, all of the agents must use the same binding settings (in other words, either the default global segmentation bindings or hierarchical port binding).



1. Edit the :ref:`Agent Configuration File`:

   .. include:: /_static/reuse/edit-agent-config-file.rst


2. Set the :ref:`heirarchical port binding settings <>` in the :ref:`L2 Segmentation Mode` section as appropriate for your environment.

   .. code-block:: console
      :caption: Hierarchical Port Binding Example
      :emphasize-lines: 9, 14, 18

      #
      f5_network_segment_physical_network = edgeswitch002ports0305
      #
      f5_network_segment_polling_interval = 10
      #
      f5_network_segment_gross_timeout = 300

\

.. important::

   You must comment out the :code:`f5_network_segment_physical_network` parameter if you're not using hierarchical port binding.


.. rubric:: Footnotes
.. [#caveat1] See :ref:`Agent Redundancy and Scale Out <lbaas-agent-redundancy>`
.. [#osvlans] `ML2 Hierarchical Port Binding specs <https://specs.openstack.org/openstack/neutron-specs/specs/kilo/ml2-hierarchical-port-binding.html#problem-description>`_.


.. _hierarchical port binding: https://specs.openstack.org/openstack/neutron-specs/specs/kilo/ml2-hierarchical-port-binding.html
.. _ML2: https://wiki.openstack.org/wiki/Neutron/ML2
.. _system configuration: https://support.f5.com/kb/en-us/products/big-ip_ltm/manuals/product/bigip-system-initial-configuration-12-0-0/2.html#conceptid
.. _local traffic management: https://support.f5.com/kb/en-us/products/big-ip_ltm/manuals/product/ltm-basics-12-0-0.html
.. _device service clustering: https://support.f5.com/kb/en-us/products/big-ip_ltm/manuals/product/bigip-device-service-clustering-admin-12-0-0.html



