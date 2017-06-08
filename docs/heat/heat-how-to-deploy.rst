.. _heat-how-to-deploy:

How to Deploy OpenStack Heat Templates
======================================

Use the instructions provided here to deploy any of the templates in F5's :ref:`heat orchestration template library <heat:templates-home>`.

OpenStack CLI
-------------

.. rubric:: Platform Requirements
- `python-openstackclient`_ configured with the ``python-heatclient`` plugin
- |heat-p|

The |heat| documentation extensively covers creation and management of Heat stacks.
Here, we provide our preferred means of defining a Heat template's required parameters and deploying it via the command line.

.. note::

   This example uses the F5-supported :ref:`deploy-lb Heat template <heat:deploy-lb>`.


#. Download the template you want to deploy from the :ref:`F5 Heat library </products/templates/f5-openstack-hot/latest/f5-supported/>`.
#. Save the definition for each of the template's required configuration parameters in an `environment file`_.

   .. literalinclude:: /_static/config_examples/deploy-lb.params.yaml
      :caption: **Example:** Environment file for the ``deploy-lb`` Heat template
      :linenos:

   Replace ``image``, ``flavor``, ``ssh-key``, ``security group``, and ``network`` with the appropriate values for your environment.


   .. tip::

      Protect your BIG-IP login information!
      Store your username and password as `environment variables`_ and reference them in your environment file.

#. Create the Heat stack.

   .. code-block:: console

      $ openstack stack create -f deploy-lb.yaml -e deploy-lb.params.yaml


OpenStack Horizon
-----------------

Follow the directions below to deploy a Heat stack using the `OpenStack Horizon`_ dashboard.

1. Go to :menuselection:`Orchestration --> Stacks`.

2. Click :guilabel:`Launch Stack`.

3. Choose the :guilabel:`Template File` from its location on your machine, then click :guilabel:`Next`.

4. Provide the information required for the Heat engine to build your stack.

5. Click :guilabel:`Launch`.

In the :guilabel:`Stacks` table, the status changes to :guilabel:`Create complete` when the stack is finished.


.. _python-openstackclient: https://docs.openstack.org/developer/python-openstackclient/
.. _environment file: https://docs.openstack.org/developer/heat/template_guide/environment.html
.. _environment variables: https://docs.openstack.org/user-guide/common/cli-set-environment-variables-using-openstack-rc.html
