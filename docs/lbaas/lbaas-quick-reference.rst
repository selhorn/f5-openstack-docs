.. _lbaas-quick-start:

F5 LBaaSv2 Quick Reference
==========================

.. include:: /_static/reuse/applies-to-ALL.rst

This reference sheet provides links to the information you need to get the |oslbaas| up and running.

#. Install the |oslbaas| packages

   - :ref:`Download and install the F5 service provider package`.
   - :ref:`Install the F5 OpenStack BIG-IP Controller`.
   - :ref:`Install the F5 OpenStack LBaaSv2 Driver`.

#. :ref:`Edit the configuration file <configure-the-f5-openstack-agent>` as appropriate for your environment.

   :ref:`Sample Configurations <agent:agent-config-examples>`

   * :ref:`Global Routed Mode <agent:grm-config-example>`
   * :ref:`GRE tunnels <agent:gre-config-example>`
   * :ref:`VxLAN tunnels <agent:vxlan-config-example>`
   * :ref:`Tagged VLANs <agent:vlan-config-example>`

   .. todo:: add section tags to agent/README sample configs
   \

   .. important::

      The Neutron configurations required may differ depending on your OpenStack platform.
      Please see our partners' documentation for more information.

      - `Hewlett Packard Enterprise <http://docs.hpcloud.com/#3.x/helion/networking/lbaas_admin.html>`_
      - `Mirantis <https://www.mirantis.com/partners/f5-networks/>`_
      - `RedHat <https://access.redhat.com/ecosystem/software/1446683>`_

#. :ref:`Set up Neutron for F5 LBaaSv2 <driver:configure-neutron-lbaasv2-driver>`.
#. :ref:`Start the F5 OpenStack BIG-IP Controller <agent:start-the-agent>`.

What's Next
-----------

- :ref:`Set up a basic load balancer using the Neutron CLI <lbaas-basic-loadbalancer>`.
- Discover how the |agent-long| :ref:`maps Neutron commands to BIG-IP objects <neutron-bigip-command-mapping>`.

.. rubric:: Footnotes
.. [#licensing] You need a Better or Best license if you plan to use GRE or VxLAN tunnels in an L2/L3-adjacent under-the-cloud deployment.

.. _license: https://f5.com/products/how-to-buy/simplified-licensing
.. _OpenStack Networking Concepts: http://docs.openstack.org/liberty/networking-guide/
.. _agent configuration file: /products/openstack/openstack-bigip-ctlr/latest/index.html#agent-configuration-file
