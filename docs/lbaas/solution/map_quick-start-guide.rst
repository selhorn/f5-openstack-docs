F5 LBaaSv2 Quick Start Guide
============================

.. rubric:: Prerequisites

- Operational OpenStack cloud (|openstack| release).
- An :term:`overcloud` or :term:`undercloud` BIG-IP :term:`device` or :term:`device cluster`.
- You must have the appropriate `license`_ for the BIG-IP features you wish to use.

  .. note:: **To use GRE or VxLAN tunnels, you need a Better or Best license.**

- Basic understanding of `OpenStack networking concepts`_.
- Basic understanding of `BIG-IP Local Traffic Manager <https://support.f5.com/kb/en-us/products/big-ip_ltm/manuals/product/ltm-basics-12-0-0.html>`_ (LTM)
- Both ``pip`` and ``git`` installed on the Neutron controller. (It may be necessary to use ``sudo`` to install the F5 LBaaS packages and dependencies, depending on your environment.)


Install the F5 Service Provider Package
---------------------------------------

.. warning:: The F5 service provider package must be installed on your Neutron controller or F5 LBaaSv2 will not work.

.. rubric:: Download the F5 LBaaSv2 service provider package and add it to the python path for ``neutron_lbaas``.

1. Download from GitHub.

.. code-block:: shell

    $ curl -O -L https://github.com/F5Networks/neutron-lbaas/releases/download/v8.0.1/f5.tgz


2. Install the service provider package on the Neutron controller.

    a. CentOS:

    .. code-block:: text

        $ sudo tar xvf f5.tgz -C /usr/lib/python2.7/site-packages/neutron_lbaas/drivers/

    b. Ubuntu:

    .. code-block:: text

        $ sudo tar xvf f5.tgz –C /usr/lib/python2.7/dist-packages/neutron_lbaas/drivers/


Install the F5 Agent
--------------------

.. topic:: To install the ``f5-openstack-agent`` package for v |release|:

    .. parsed-literal::

        $ sudo pip install |f5_agent_pip_url|

    .. tip::

        See the :ref:`F5 Agent documentation <agent:home>` for rpm and dpkg installation instructions.


Install the F5 LBaaSv2 Driver
-----------------------------

.. include:: includes/topic_install-f5-lbaasv2-driver.rst
    :start-line: 5


.. seealso:: :ref:`Basic Environment Requirements for F5 LBaaSv2`




Configure F5 LBaaSv2
====================

.. include:: includes/topic_config-agent-overview.rst
    :start-line: 4

The table below contains a summary of the recommended F5 LBaaSv2 :ref:`configuration settings <Configure the F5 OpenStack Agent>`.

.. note:: This table is not a comprehensive list of all available options. For additional information, and to view all available configuration options, please see :ref:`Supported Features`.

.. include:: includes/ref_agent-config-settings-table.rst
    :start-line: 5

.. include:: includes/ref_agent-config-file.rst
    :start-after: each available configuration option.
    :end-before: :ref:`Global Routed Mode

* :ref:`Global Routed Mode` :download:`f5-openstack-agent.grm.ini <_static/f5-openstack-agent.grm.ini>`

* GRE tunnels :download:`f5-openstack-agent.gre.ini <_static/f5-openstack-agent.gre.ini>`

* VxLAN tunnels :download:`f5-openstack-agent.vxlan.ini <_static/f5-openstack-agent.vxlan.ini>`

* Tagged VLANs (without tunnels) :download:`f5-openstack-agent.vlan.ini <_static/f5-openstack-agent.vlan.ini>`


.. include:: includes/topic_configure-neutron-lbaasv2.rst
    :start-line: 4

.. important::

    The Neutron configurations required may differ depending on your OS. Please see our partners' documentation for more information.

    - `Hewlett Packard Enterprise <http://docs.hpcloud.com/#3.x/helion/networking/lbaas_admin.html>`_
    - `Mirantis <https://www.mirantis.com/partners/f5-networks/>`_
    - `RedHat <https://access.redhat.com/ecosystem/software/1446683>`_
    

.. include:: includes/topic_start-f5-agent.rst
    :start-line: 4

Next Steps
==========

- See the :ref:`Coding Example` for the commands to use to configure basic load balancing via the Neutron CLI.
- See :ref:`F5 LBaaSv2 to BIG-IP Configuration Mapping` to discover what the F5 agent configures on the BIG-IP.


.. _license: https://f5.com/products/how-to-buy/simplified-licensing
.. _OpenStack Networking Concepts: http://docs.openstack.org/liberty/networking-guide/

