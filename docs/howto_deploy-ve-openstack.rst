.. _deploy_big-ip_openstack:

How To Deploy BIG-IP Virtual Edition in OpenStack
=================================================

.. _ve-deploy-guide-before-you-begin:

Before you begin
----------------

You must have the following before deploying a BIG-IP VE in OpenStack:

- A functional OpenStack environment with at least one controller node, one compute node, and one network node.

  .. seealso::

     - `OpenStack End User Guide <http://docs.openstack.org/user-guide/index.html>`_
     - :ref:`OpenStack Platform Partnerships`

- Basic understanding of `OpenStack networking concepts`_.

.. important::

   You need to source a credentials file with admin permissions (e.g., ``source keystonerc_admin``) to use the ``openstack``, ``nova``, and ``neutron`` commands.

   You can also make configurations via the OpenStack dashboard.
   See the `OpenStack dashboard user guide <http://docs.openstack.org/user-guide/dashboard.html>`_ for more information.

.. _ve-initial-setup:

.. _topic_deploy-ve-initial-setup:

Initial Setup
-------------

Projects, Roles, and Users
``````````````````````````

You can create any number of projects (also called tenants), roles, and users in OpenStack to suit your needs. At minimum, for the purposes of this guide, you'll need to create an admin project and role; then, create an admin user and assign to it the admin role and project.

.. topic:: Example

   .. code-block:: shell

      $ sudo openstack project create admin
      $ sudo openstack role create admin
      $ sudo openstack user create admin --project=admin --password=default --email=<email_address> --role=admin

Security Groups
```````````````

The security groups below will set up the necessary rules for communication between OpenStack tenants and BIG-IP.

.. tip::

   :ref:`F5 OpenStack Heat <heat:home>` contains templates that can create these security groups for you.
   See the :ref:`Heat User Guide <heat:heat-user-guide>` for more information about using F5's templates to launch Heat stacks.

1. Create the ``BIG-IP_default`` security group.

   .. code-block:: shell

      $ sudo neutron security-group-create BIG-IP_default

2. Add incoming traffic policies to the security group, as needed.

   * ICMP

   .. code-block:: shell

      $ sudo neutron security-group-rule-create --protocol icmp --direction ingress BIG-IP_default
      Created a new security_group_rule:
      +-------------------+--------------------------------------+
      | Field             | Value                                |
      +-------------------+--------------------------------------+
      | direction         | ingress                              |
      | ethertype         | IPv4                                 |
      | id                | e589ab98-7358-41e0-988e-e54ef3b7e445 |
      | port_range_max    |                                      |
      | port_range_min    |                                      |
      | protocol          | icmp                                 |
      | remote_group_id   |                                      |
      | remote_ip_prefix  |                                      |
      | security_group_id | ea8c4843-3704-444d-a5fe-17d5a60261fd |
      | tenant_id         | 1a35d6558b59423e83f4500f1ebc1cec     |
      +-------------------+--------------------------------------+

   * SSH

   .. code-block:: shell

      $ sudo neutron security-group-rule-create --protocol tcp --port-range-min 22 --port-range-max 22 --direction ingress BIG-IP_default
      Created a new security_group_rule:
      +-------------------+--------------------------------------+
      | Field             | Value                                |
      +-------------------+--------------------------------------+
      | direction         | ingress                              |
      | ethertype         | IPv4                                 |
      | id                | 6064fdaf-df1f-4924-b6aa-5af9c33d31f5 |
      | port_range_max    | 22                                   |
      | port_range_min    | 22                                   |
      | protocol          | tcp                                  |
      | remote_group_id   |                                      |
      | remote_ip_prefix  |                                      |
      | security_group_id | ea8c4843-3704-444d-a5fe-17d5a60261fd |
      | tenant_id         | 1a35d6558b59423e83f4500f1ebc1cec     |
      +-------------------+--------------------------------------+

   * HTTP

   .. code-block:: shell

      $ sudo neutron security-group-rule-create --protocol tcp --port-range-min 80 --port-range-max 80 --direction ingress BIG-IP_default
      Created a new security_group_rule:
      +-------------------+--------------------------------------+
      | Field             | Value                                |
      +-------------------+--------------------------------------+
      | direction         | ingress                              |
      | ethertype         | IPv4                                 |
      | id                | df34ddf2-8a63-4772-aee8-6a688f3bf0dc |
      | port_range_max    | 80                                   |
      | port_range_min    | 80                                   |
      | protocol          | tcp                                  |
      | remote_group_id   |                                      |
      | remote_ip_prefix  |                                      |
      | security_group_id | ea8c4843-3704-444d-a5fe-17d5a60261fd |
      | tenant_id         | 1a35d6558b59423e83f4500f1ebc1cec     |
      +-------------------+--------------------------------------+

   * SSL

   .. code-block:: shell

      $ sudo neutron security-group-rule-create --protocol tcp --port-range-min 443 --port-range-max 443 --direction ingress BIG-IP_default
      Created a new security_group_rule:
      +-------------------+--------------------------------------+
      | Field             | Value                                |
      +-------------------+--------------------------------------+
      | direction         | ingress                              |
      | ethertype         | IPv4                                 |
      | id                | 9cda1fcc-c403-4523-9c36-2ff0b4b0dbd8 |
      | port_range_max    | 443                                  |
      | port_range_min    | 443                                  |
      | protocol          | tcp                                  |
      | remote_group_id   |                                      |
      | remote_ip_prefix  |                                      |
      | security_group_id | ea8c4843-3704-444d-a5fe-17d5a60261fd |
      | tenant_id         | 1a35d6558b59423e83f4500f1ebc1cec     |
      +-------------------+--------------------------------------+

   * VXLAN

   .. code-block:: shell

      $ sudo neutron security-group-rule-create --protocol udp --port-range-min 4789 --port-range-max 4789 --direction ingress BIG-IP_default
      Created a new security_group_rule:
      +-------------------+--------------------------------------+
      | Field             | Value                                |
      +-------------------+--------------------------------------+
      | direction         | ingress                              |
      | ethertype         | IPv4                                 |
      | id                | 44236cb0-2f9e-4e5f-8035-f97275ceed15 |
      | port_range_max    | 4789                                 |
      | port_range_min    | 4789                                 |
      | protocol          | udp                                  |
      | remote_group_id   |                                      |
      | remote_ip_prefix  |                                      |
      | security_group_id | ea8c4843-3704-444d-a5fe-17d5a60261fd |
      | tenant_id         | 1a35d6558b59423e83f4500f1ebc1cec     |
      +-------------------+--------------------------------------+

   * GRE

   .. code-block:: shell

      $ sudo neutron security-group-rule-create --protocol 47 --direction ingress BIG-IP_default
      Created a new security_group_rule:
      +-------------------+--------------------------------------+
      | Field             | Value                                |
      +-------------------+--------------------------------------+
      | direction         | ingress                              |
      | ethertype         | IPv4                                 |
      | id                | e12dbdb2-e88b-4dd7-9f6c-3515f51db9af |
      | port_range_max    |                                      |
      | port_range_min    |                                      |
      | protocol          | 47                                   |
      | remote_group_id   |                                      |
      | remote_ip_prefix  |                                      |
      | security_group_id | ea8c4843-3704-444d-a5fe-17d5a60261fd |
      | tenant_id         | 1a35d6558b59423e83f4500f1ebc1cec     |
      +-------------------+--------------------------------------+

Package Information
```````````````````

BIG-IP needs to be able to detect that it’s running on a VM.
Check :file:`/etc/nova/release` to make sure that the vendor, product, and package information is stored there.

.. code-block:: shell

   $ cat /etc/nova/release
   [Nova]
   vendor = Fedora Project
   product = OpenStack Nova
   package = 1.el7

If the package information isn't present, enter the appropriate information for your environment.

.. code-block:: shell

   $ echo -e "[Nova]\nvendor = Fedora Project\nproduct = OpenStack Nova\npackage = 1.el7" > /etc/nova/release

Custom Flavors
``````````````

While the built-in `Nova Flavors <http://docs.openstack.org/admin-guide/compute-flavors.html>`_ can be used with BIG-IP VE, you can also create your own custom flavors.

.. tip::

   For information regarding BIG-IP® VE image sizes and minimum requirements, see the :ref:`BIG-IP VE Flavor Requirements <big-ip_flavors>`.

To define a custom flavor via the command line:

.. code-block:: shell

   flavor_id=$(cat /proc/sys/kernel/random/uuid) nova flavor-create f5small $flavor_id 4096 20 2

You can also create new custom flavors via the OpenStack dashboard.
To do so, go to :menuselection:`System --> Flavors` and click :guilabel:`Create Flavor`.

Restart
```````

Once your setup is complete, restart the Nova-Compute service:

.. code-block:: shell

   $ sudo service nova-compute restart // Debian/Ubuntu
   $ sudo systemctl restart nova-compute // Redhat/CentOS


.. _launch-bigip-ve-gui:

Launch BIG-IP Virtual Edition using OpenStack Horizon
-----------------------------------------------------

.. _deploy-bigip-overview:

Overview
````````

We recommend using the :ref:`F5 Openstack Heat <heat:home>` templates to deploy a BIG-IP Virtual Edition (VE) in OpenStack. The :ref:`F5-supported Heat templates <#tbd>` can deploy a number of common-use-case stacks.


.. _import-ve-image:

Patch and Import a BIG-IP image
```````````````````````````````

Use the F5-supported :ref:`image prep and upload` Heat orchestration template to import a standard VE image and patch it for use in OpenStack.


.. _launch_big-ip_instance:

Launch an Instance
``````````````````

To launch a BIG-IP instance using the OpenStack dashboard:

1. Go to ``http://<ip_address>/dashboard`` and log in with your admin credentials.

2. Go to :menuselection:`Project --> Compute --> Instances`, then click :guilabel:`Launch Instance`.

   -  On the :guilabel:`Project & User` tab:

      - select :guilabel:`admin` for each.

   -  On the :guilabel:`Details` tab:

      - enter a descriptive instance name;
      - choose your custom flavor;
      - select :guilabel:`boot from image` as the boot source;
      - select your BIG-IP image.

   -  On the :guilabel:`Access & Security` tab:

      - select the :guilabel:`BIG-IP_default` security group.

   -  On the :guilabel:`Network` tab:

      - select networks as appropriate (at least two).

   -  Click :guilabel:`Launch`.

.. warning::

   Do not select the physical external network when launching an instance.
   Choose the :ref:`VLANs <concept_vlans>` you set up for use with your BIG-IP VLANs.

.. _assign-floating-ip:

Assign a Floating IP Address
````````````````````````````

Use the OpenStack dashboard to assign a floating IP address to the instance.

1. Go to :menuselection:`Project --> Compute --> Instances`, then choose :guilabel:`Associate Floating IP` from the drop-down menu in the :menuselection:`Actions` column.

2. Select a :guilabel:`Floating IP` from the :guilabel:`IP Address` drop-down menu.

3. In the :guilabel:`port` drop-down, select the port for your BIG-IP® image that corresponds to the external VLAN you set up for your BIG-IP®.

4. Click :guilabel:`Associate`.

.. tip::

   If no floating IP addresses are available, click ``+`` to generate one, then click :guilabel:`Allocate`.










