.. _lbaas-differentiated-service-env:

Differentiated Service Environments
===================================

Overview
--------

The |oslbaas| can manage multiple distinct BIG-IP environments, called "differentiated service environments".
A :dfn:`differentiated service environment` is a uniquely-named environment that has:

- a dedicated |driver| instance,
- a dedicated messaging queue, and
- a dedicated |agent| instance.

.. note::

   You can use multiple |agent| instances in a differentiated service environment.
   Each agent must manage a separate BIG-IP device

The |driver| instance for a differentiated service environment assigns LBaaS tasks to |agent-long| instances running in that service environment.

The default service environment prefix, :code:`Project`, corresponds to the generic F5 Networks :ref:`service provider driver <Set 'F5Networks' as the LBaaSv2 Service Provider>` entry in the Neutron LBaaS configuration file (:file:`/etc/neutron/neutron_lbaas.conf`).

A custom service environment prefix

.. important::

   Each unique service environment must have a corresponding Neutron service provider driver.
   Use the :ref:`F5 environment generator`, which is built into the |driver-long|, to create a new service environment and configure Neutron to use it.

Use Case
````````

You can use differentiated service environments to manage LBaaS objects for unique environments (for example, "dev", "prod", "test", etc.).
When used with :ref:`capacity-based scale out`, differentiated service environments provide redundancy and scale out for the |agent-long|.

Caveats
```````

- Differentiated service environments aren't compatible with `Virtual Clustered Multiprocessing`_ (vCMP) systems.
  BIG-IP devices cannot share data or resources across differentiated service environments; this precludes the use of vCMP because vCMP guests share global VLAN IDs.


Create a new F5 service provider driver
---------------------------------------

Use the :ref:`F5 environment generator` to automatically create and set up a new |driver| instance.

.. code-block:: console

   add_f5agent_environment <env_name>

.. tip::

   The environment name field must be eight (8) characters or less.

Configure the |agent-long|
``````````````````````````

#. :ref:`Edit the agent configuration file`.

#. Change the ``environment_prefix`` parameter to match the name of your custom environment.

#. Restart Neutron.

   .. include:: /_static/reuse/restart-neutron.rst

#. Start the |agent-long|.

   .. include:: /_static/reuse/start-f5-agent.rst




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

   - :ref:`F5 OpenStack BIG-IP Controller Redundancy and Scale-out <lbaas-agent-redundancy>`

.. _Virtual Clustered Microprocessing: https://support.f5.com/kb/en-us/products/big-ip_ltm/manuals/product/vcmp-administration-appliances-12-1-1/1.html
