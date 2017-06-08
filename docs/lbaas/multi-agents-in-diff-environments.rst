Multiple Agents and Differentiated Service Environments
=======================================================

Overview
--------

You can run :ref:`multiple F5 agents <Agent Redundancy and Scale Out>` on separate hosts in OpenStack to provide agent redundancy and scale out.
Additionally, you can set up custom :ref:`service environments <Differentiated Service Environments>` in your OpenStack cloud to manage environments with different requirements and/or configurations.

Use Case
--------

Normally, if you run more than one |agent-long| on your Neutron controller, each |agent| instance must manage a unique BIG-IP device or :term:`cluster`.
In this scenario, you can't designate which |agent| instance/BIG-IP device you want to create a new load balancer for.
Rather, the |driver-long| will assign the LBaaS task to the first available agent it finds.

For a higher degree of flexibility and control, you can create a custom :ref:`service environment <Differentiated Service Environments>`.
In a custom service environment, you can manage a single BIG-IP device or cluster with multiple |agent| instances.

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
   - :ref:`Agent Redundancy and Scale-out <lbaas-agent-redundancy>`
