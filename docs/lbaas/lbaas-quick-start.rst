.. _lbaas-quick-start:

F5 LBaaSv2 Quick Start Guide
============================

.. sidebar:: Applies to:

   OpenStack |openstack|

.. important::

   Depending on your account permissions and environment, you might need to use :code:`sudo` to complete any of the steps outlined here.


Prerequisites
-------------

- A BIG-IP :term:`device` or :Term:`device cluster` provisioned with the appropriate `license`_ for the features you want to use. [#licensing]_
- Both ``pip`` and ``git`` installed on the Neutron controller.

.. seealso::

   :ref:`Basic Environment Requirements for F5 LBaaSv2`


Install the |oslbaas| packages
------------------------------

.. danger::

   If you don't install the F5 service provider package on your Neutron controller, the F5 LBaaSv2 Integration won't work.

#. Download and install the F5 service provider package.

   .. code-block:: shell

      curl -O -L https://github.com/F5Networks/neutron-lbaas/releases/download/v8.0.1/f5.tgz
      sudo tar xvf f5.tgz -C /usr/lib/python2.7/site-packages/neutron_lbaas/drivers/            \\ CentOS
      sudo tar xvf f5.tgz –C /usr/lib/python2.7/dist-packages/neutron_lbaas/drivers/            \\ Ubuntu


#. Install the |agent| package.

   .. parsed-literal::

      sudo pip install |f5_agent_pip_url|

   .. tip::

      See the `F5 OpenStack BIG-IP Controller`_ documentation for ``rpm`` and ``dpkg`` downloads and installation instructions.


#. Install the |driver| package.

   .. parsed-literal::

      sudo pip install |f5_driver_pip_url|


Set up the F5 BIG-IP Controller
-------------------------------

.. tip::

   The |agent| `agent configuration file`_ tells the |agent-long| :

   * how to communicate with your BIG-IP device(s) and
   * what your existing BIG-IP device settings are.

#. Edit the |agent| configuration file as appropriate for your environment.

   .. code-block:: bash

      vi /etc/neutron/services/f5/f5-openstack-agent.ini

:ref:`Sample Configurations <agent:agent-config-examples>`

* :ref:`Global Routed Mode <agent:grm-config-example>`
* :ref:`GRE tunnels <agent:gre-config-example>`
* :ref:`VxLAN tunnels <agent:vxlan-config-example>`
* :ref:`Tagged VLANs <agent:vlan-config-example>`

.. todo:: add section tags to agent/README sample configs

.. important::

   The Neutron configurations required may differ depending on your OpenStack platform.
   Please see our partners' documentation for more information.

   - `Hewlett Packard Enterprise <http://docs.hpcloud.com/#3.x/helion/networking/lbaas_admin.html>`_
   - `Mirantis <https://www.mirantis.com/partners/f5-networks/>`_
   - `RedHat <https://access.redhat.com/ecosystem/software/1446683>`_
    

include:: includes/topic_start-f5-agent.rst


What's Next
-----------

- :ref:`Set up a basic load balancer <lbaas-basic-loadbalancer>` via the Neutron CLI.
- See :ref:`F5 LBaaSv2 to BIG-IP Configuration Mapping`.


.. rubric:: Footnotes
.. [#licensing] You need a Better or Best license if you plan to use GRE or VxLAN tunnels in an L2/L3-adjacent under-the-cloud deployment.

.. _license: https://f5.com/products/how-to-buy/simplified-licensing
.. _OpenStack Networking Concepts: http://docs.openstack.org/liberty/networking-guide/
.. _agent configuration file: /products/openstack/openstack-bigip-ctlr/latest/index.html#agent-configuration-file
