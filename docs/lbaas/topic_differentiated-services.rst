.. _lbaas-differentiated-service-env:

Differentiated Service Environments
===================================

Overview
--------

The |oslbaas| can manage multiple distinct BIG-IP environments, called :dfn:`differentiated service environment`.
A differentiated service environment is a uniquely-named environment with

- a dedicated |driver| instance,
- a dedicated messaging queue, and
- a dedicated |agent| 

for which dedicated F5 LBaaS services are required -- a unique |driver-long| instance has its own named messaging queue.
The |driver| instance assigns LBaaS tasks for that environment assign tasks to |agent-long| instances running in that service environment.

The ``environment_prefix`` parameter in the :ref:`agent configuration file` defines the |agent| service environment.
When you create a new ``lbaas-loadbalancer`` in OpenStack, the environment prefix is prepended to the OpenStack tenant id and used to create a new partition on your BIG-IP device(s).
The default ``environment_prefix`` parameter is ``Project``.

You can use differentiated service environments with :ref:`capacity-based scale out` to provide agent redundancy and scale out across BIG-IP device groups.

Neutron Service Provider Driver Entries
```````````````````````````````````````

The default service environment, ``Project``, corresponds to the generic F5Networks :ref:`service provider driver <Set 'F5Networks' as the LBaaSv2 Service Provider>` entry in the Neutron LBaaS configuration file (:file:`/etc/neutron/neutron_lbaas.conf`).

.. important::

   Each unique service environment must have a corresponding service provider driver entry.
   You can use the :ref:`F5 environment generator` to easily create a new environment and configure Neutron to use it.

Use Case
--------

Differentiated service environments can be used to manage LBaaS objects for unique environments, which may have requirements that differ from those of other service environments.

Prerequisites
-------------

- Licensed, operational BIG-IP :term:`device` or :term:`device service cluster`.
- Operational OpenStack cloud (|openstack| release).
- F5 :ref:`agent <Install the F5 Agent>` and :ref:`LBaaSv2 driver <Install the F5 LBaaSv2 Driver>` installed on all hosts from which BIG-IP services will be provisioned.
- Basic understanding of `BIG-IP system configuration <https://support.f5.com/kb/en-us/products/big-ip_ltm/manuals/product/bigip-system-initial-configuration-12-0-0/2.html#conceptid>`_.
- Basic understanding of `BIG-IP Local Traffic Management <https://support.f5.com/kb/en-us/products/big-ip_ltm/manuals/product/ltm-basics-12-0-0.html>`_

Caveats
-------

- BIG-IP devices can not share anything across differentiated service environments. This precludes the use of vCMP, because vCMP guests share global VLAN IDs.


Configuration
-------------

Create a Service Provider Driver
````````````````````````````````

You can use the :ref:`F5 environment generator` to automatically generate, and configure Neutron to use, a new service provider driver for a custom environment. On each Neutron controller which will host your custom environment, run the following command:

    .. code-block:: shell

        $ sudo add_f5agent_environment <env_name>

The environment name is limited to 8 characters in length.

Configure the |agent-long|
``````````````````````````

#. :ref:`Edit the agent configuration file`.

#. Change the ``environment_prefix`` parameter to match the name of your custom environment.

#. Restart Neutron.

   .. include:: /_static/reuse/restart-neutron.rst

#. Start the |agent-long|.

   .. include:: /_static/reuse/start-f5-agent.rst



Further Reading
---------------

.. seealso::

    * :ref:`Configure the F5 OpenStack Agent`
    * :ref:`Configure Neutron for LBaaSv2`
    * :ref:`F5 Environment Generator`

-------------------------------------------------------------------------------

You can set up custom :ref:`service environments <Differentiated Service Environments>` in your OpenStack cloud to manage projects with different requirements and/or configurations.
If you're using multiple service environments, you can designate which |agent| instance(s) should handle LBaaS tasks for each environment.

Use Case
--------

Normally, if you run more than one |agent-long| on your Neutron controller, each |agent| instance must manage a separate BIG-IP device or :term:`cluster`.
In this scenario, when you create a new load balancer, you can't choose which |agent| instance (and, therefore, which BIG-IP device) you want to use.
Instead, the |driver-long| assign the LBaaS task to the first available |agent| instance it finds.

For a higher degree of flexibility and control, you can create a custom :ref:`service environment <Differentiated Service Environments>`.
In a custom service environment, you can manage a single BIG-IP device or cluster with multiple |agent| instances.
This allows you to control which |agent-long| handles specific requests,

Prerequisites
-------------

- F5 :ref:`Openstack BIG-IP Controller <Install the F5 Agent>` and :ref:`LBaaSv2 driver <Install the F5 LBaaSv2 Driver>` installed on the Neutron controller.
- The |agent-long| installed on two (2) or more unique hosts.

Caveats
-------

- Each |agent| instance must run on a unique host.
  This could be the Neutron controller, a compute node, a container, a virtual machine, etc.

Configuration
-------------

#. :ref:`Generate a new custom environment <driver:environment-generator>` on the Neutron controller.

   .. code-block:: console
      :caption: Example: Add a custom environment called "dev1"

      add_f5agent_environment dev1

#. Edit the |agent| configuration file:

   - Replace the default ``environment_prefix`` with the name of the new custom environment.

     .. code-block:: console

        vi /etc/neutron/services/f5/f5-openstack-agent.ini
        #
        # environment_prefix = 'dev1'
        #

   - Add/update the iControl endpoints, as needed.

     .. code-block:: console

        #
        icontrol_hostname = 1.2.3.4, 5.6.7.8
        #
        ...
        #
        icontrol_username = <username>
        ...
        #
        icontrol_password = <password>
        #

   - Save the file with a new name.

     .. code-block:: console
        :caption: Example

        :w f5-openstack-agent_dev1.ini


#. Copy the |agent|, Neutron, and Neutron LBaaS configuration files from the Neutron controller to each host on which you want to run an |agent| instance.

   .. code-block:: console

      cp /etc/neutron/services/f5/f5-openstack-agent_dev1.ini <hostname>:/etc/neutron/services/f5/f5-openstack-agent_dev1.ini
      cp /etc/neutron/neutron.conf <hostname>:/etc/neutron/neutron.conf
      cp /etc/neutron/neutron_lbaas.conf <hostname>:/etc/neutron/neutron_lbaas.conf

#. Restart Neutron.

   .. include:: /_static/reuse/restart-neutron.rst

#. Start the |agent-long| on each host.

   .. include:: /_static/reuse/start-f5-agent.rst

Usage
-----

When you create a new load balancer, you must specify the service provider driver to use.
This is how the |driver-long| knows which queue should receive the task (in other words, on which BIG-IP it should add the new partition).

.. rubric:: Example:

.. code-block:: console

   (neutron) lbaas-loadbalancer-create --name lb_dev1 --provider dev1 b3fa44a0-3187-4a49-853a-24819bc24d3e
   Created a new loadbalancer:
   +---------------------+--------------------------------------+
   | Field               | Value                                |
   +---------------------+--------------------------------------+
   | admin_state_up      | True                                 |
   | description         |                                      |
   | id                  | fcd874ce-6dad-4aef-9e69-98d1590738cd |
   | listeners           |                                      |
   | name                | lb_dev1                              |
   | operating_status    | OFFLINE                              |
   | provider            | dev1                                 |
   | provisioning_status | PENDING_CREATE                       |
   | tenant_id           | 1b2b505dafbc487fb805c6c9de9459a7     |
   | vip_address         | 10.1.2.7                             |
   | vip_port_id         | 079eb9e5-dc63-4dbf-bc15-f38f5fdeee92 |
   | vip_subnet_id       | b3fa44a0-3187-4a49-853a-24819bc24d3e |
   +---------------------+--------------------------------------+


.. seealso::

   - :ref:`Differentiated Service Environments <lbaas-differentiated-service-env>`
   - :ref:`F5 OpenStack BIG-IP Controller Redundancy and Scale-out <lbaas-agent-redundancy>`
