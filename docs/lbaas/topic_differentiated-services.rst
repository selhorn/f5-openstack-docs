.. _lbaas-differentiated-service-env:

Manage BIG-IP devices using Differentiated Service Environments
===============================================================

Overview
--------

The |oslbaas| can manage multiple distinct BIG-IP environments, called "differentiated service environments".
A :dfn:`differentiated service environment` is a uniquely-named environment that has:

- a dedicated |driver| instance,
- a dedicated messaging queue, and
- a dedicated |agent| instance.

.. note::

   - You can use differentiated service environments to manage the same BIG-IP device or cluster with multiple |agent| instances.
     In a multiple-agent setup, each |agent| instance manages a distinct environment that corresponds to a specific BIG-IP partition.

   - The :ref:`F5 environment generator`, a tool built in to the |driver-long|, creates new service environments for you and configures Neutron to use the new service provider drivers.

   - Differentiated service environments aren't compatible with `Virtual Clustered Multiprocessing`_ (vCMP) systems.
     BIG-IP devices cannot share data or resources across differentiated service environments; this precludes the use of vCMP because vCMP guests share global VLAN IDs


Create a new F5 service provider driver
---------------------------------------

:ref:`Generate a new custom environment <driver:environment-generator>` on the Neutron controller.

   .. code-block:: console
      :caption: Example: Add a custom environment called "dev1"

      add_f5agent_environment dev1

.. tip::

   The environment name field must be eight (8) characters or less.


Configure the |agent-long|
--------------------------

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
        icontrol_username = myusername
        ...
        #
        icontrol_password = mypassword
        #

   - Save the file with a new name.

     .. code-block:: console
        :caption: Example

        :w f5-openstack-agent_dev1.ini


#. Set up additional hosts. [#multihost]_


   Copy the |agent|, Neutron, and Neutron LBaaS configuration files from the Neutron controller to each host on which you want to run an |agent| instance.

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

Specify the service provider driver to use when you create a new load balancer.
This determines which |driver| messaging queue receives the task.

.. tip::

   If you're using custom service environments to manage different BIG-IP devices or clusters, specifying the service provider driver lets you identify the BIG-IP device on which you want to create the new partition.


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

.. todo:: come up with a better title for this

Deep dive
---------

The default service environment prefix, :code:`Project`, corresponds to the generic "F5Networks" LBaaSv2 :ref:`service provider driver <Set 'F5Networks' as the LBaaSv2 Service Provider>` entry in the Neutron LBaaS configuration file (:file:`/etc/neutron/neutron_lbaas.conf`).
Each custom service environment (for example, "dev", "prod", "test", etc.) has a corresponding service provider driver entry in the :file:`neutron_lbaas.conf` file. When you issue a :code:`neutron lbaas-loadbalancer-create` command referencing the service provider driver for a specific environment, that |driver| instance will receive the task in its dedicated messaging queue; the |driver| instance will then assign the task to an |agent| instance in the same environment group as the driver.

Use Case
````````

When used with :ref:`capacity-based scale out`, differentiated service environments provide redundancy and scale out for the |agent-long|.
Using differentiated service environments allows you to run multiple |agent| instances on the same host to manage the same BIG-IP device.
Each unique environment corresponds to a separate BIG-IP partition; the |driver-long| for that environment assigns tasks to its associated |agent| instance, which configures objects in the environment's partition on the BIG-IP device.
This allows you to specify which |agent| instance should handle LBaaS tasks, instead of the "first-available" method the |driver-long| uses in the default environment.




.. seealso::

   - :ref:`F5 OpenStack BIG-IP Controller Redundancy and Scale-out <lbaas-agent-redundancy>`

.. _Virtual Clustered Microprocessing: https://support.f5.com/kb/en-us/products/big-ip_ltm/manuals/product/vcmp-administration-appliances-12-1-1/1.html
