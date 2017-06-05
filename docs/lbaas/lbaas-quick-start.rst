F5 LBaaSv2 Quick Start Guide
============================

.. sidebar:: Applies to:

   OpenStack |openstack|


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

includes/topic_config-agent-overview.rst


The table below contains a summary of the recommended F5 LBaaSv2 :ref:`configuration settings <Configure the F5 OpenStack Agent>`.

.. note:: This table is not a comprehensive list of all available options. For additional information, and to view all available configuration options, please see :ref:`Supported Features`.

include:: includes/ref_agent-config-settings-table.rst


include:: includes/ref_agent-config-file.rst
    :start-after: each available configuration option.
    :end-before: :ref:`Global Routed Mode

* :ref:`Global Routed Mode` f5-openstack-agent.grm.ini <_static/f5-openstack-agent.grm.ini>

* GRE tunnels f5-openstack-agent.gre.ini <agent:f5-openstack-agent.gre.ini>

* VxLAN tunnels f5-openstack-agent.vxlan.ini <_static/f5-openstack-agent.vxlan.ini>

* Tagged VLANs (without tunnels) f5-openstack-agent.vlan.ini <_static/f5-openstack-agent.vlan.ini>


include:: includes/topic_configure-neutron-lbaasv2.rst

.. important::

    The Neutron configurations required may differ depending on your OS. Please see our partners' documentation for more information.

    - `Hewlett Packard Enterprise <http://docs.hpcloud.com/#3.x/helion/networking/lbaas_admin.html>`_
    - `Mirantis <https://www.mirantis.com/partners/f5-networks/>`_
    - `RedHat <https://access.redhat.com/ecosystem/software/1446683>`_
    

include:: includes/topic_start-f5-agent.rst


What's Next
-----------

- See the :ref:`Coding Example` for the commands to use to configure basic load balancing via the Neutron CLI.
- See :ref:`F5 LBaaSv2 to BIG-IP Configuration Mapping` to discover what the F5 agent configures on the BIG-IP.


.. rubric:: Footnotes
.. [#licensing] You need a Better or Best license if you plan to use GRE or VxLAN tunnels in an L2/L3-adjacent under-the-cloud deployment.

.. _license: https://f5.com/products/how-to-buy/simplified-licensing
.. _OpenStack Networking Concepts: http://docs.openstack.org/liberty/networking-guide/

