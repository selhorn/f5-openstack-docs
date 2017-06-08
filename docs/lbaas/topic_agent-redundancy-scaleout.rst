.. _lbaas-agent-redundancy:

Redundancy and Scale Out
========================

When the Neutron LBaaS plugin loads the |driver-long|, it creates a global messaging queue.
The |agent-long| sends all callbacks and status updates to this global queue.
The |driver-long| picks up requests from the global messaging queue in a round-robin fashion, then assigns the tasks to an available |agent| instance based on "agent-tenant affinity".

Use Case
--------

You can run multiple |agent| instances **on different hosts** in your OpenStack cloud to provide redundancy and scale-out. Managing the same BIG-IP device or cluster from different hosts ensures that if one host goes down, the F5 LBaaSv2 processes remain alive and functional. It also allows you to spread the request load for the environment across multiple agents.

You can run multiple |agent-long|  on the same host only if they are each managing a different BIG-IP device or group.


Prerequisites
-------------

- Licensed, operational BIG-IP :term:`device` or :term:`device cluster`.
- Operational OpenStack cloud (|openstack| release).
- Administrator access to both the BIG-IP device(s) and the OpenStack cloud.
- All hosts running F5 LBaaSv2 must have the Neutron and Neutron LBaaS packages installed.
- All hosts running F5 LBaaSv2 must use the same Neutron database.


Caveats
-------

- You **can not** run multiple agents on the same host if they are expected to manage the same BIG-IP device or :term:`cluster`. See :ref:`Differentiated Service Environments` for information about running more than one |agent-long| /driver on the same host.
- In the standard multi-agent deployment, specifying the |agent-long| /BIG-IP pair to use when creating a new load balancer is not supported. Instead, use a custom environment as described in :ref:`Multiple Agents and Differentiated Service Environments`.


Configuration
-------------

To manage one BIG-IP device or device service group with multiple |agent-long|s, deploy F5 LBaaSv2 on separate hosts using the instructions provided below.

#. Copy the Neutron config file from your Neutron controller to each host on which you will run F5 LBaaSv2:

    .. code-block:: bash

        $ sudo cp /etc/neutron/neutron.conf <openstack_host>:/etc/neutron/neutron.conf

#. :ref:`Install the F5 Agent` and :ref:`service provider driver <Install the F5 LBaaSv2 Driver>` on each host.

#. :ref:`Configure the F5 agent <Configure the F5 OpenStack Agent>` on each host.

    .. tip::

        * Be sure to provide the iControl endpoints for all BIG-IP devices you'd like the agents to manage.
        * You can configure the |agent-long| once, on the Neutron controller, then copy the agent config file (:file:`/etc/neutron/services/f5/f5-openstack-agent.ini`) over to the other hosts.

#. Start the |agent-long|.

   .. include:: /_static/reuse/start-f5-agent.rst



Further Reading
---------------

.. seealso::

    * :ref:`Configure the F5 OpenStack Agent`
    * :ref:`Manage BIG-IP Clusters with F5 LBaaSv2`
    * :ref:`Manage Multi-Tenant BIG-IP Devices with F5 LBaaSv2`
    * :ref:`Differentiated Service Environments`
    * :ref:`Multiple Agents and Differentiated Service Environments`

.. rubric:: Footnotes
.. [#] **F5 Networks does not provide support for container service deployments.**
       If you are already well versed in containerized environments, you can run one |agent-long| per container. The neutron.conf file must be present in the container. The service provider driver does not need to run in the container; rather, it only needs to be in the container's build context.

