.. _configure-the-f5-openstack-agent:

Configure the F5 OpenStack Agent
================================

Edit the Agent Configuration File
---------------------------------

Use your text editor of choice to edit the :ref:`agent configuration file` as appropriate for your environment.

.. topic:: Example:

    .. code-block:: text

        $ sudo vi /etc/neutron/services/f5/f5-openstack-agent.ini

.. _topic-start-the-agent:

Start the F5 agent
------------------

Once you have configured the F5 agent as appropriate for your environment, use the command(s) appropriate for your OS to start the agent.

.. topic:: CentOS

    .. code-block:: text

        $ sudo systemctl enable f5-openstack-agent
        $ sudo systemctl start f5-openstack-agent

.. topic:: Ubuntu


    .. code-block:: text

        $ sudo service f5-oslbaasv2-agent start


Stop the F5 agent
-----------------


If you need to stop the F5 agent, run the command appropriate for your OS.

.. topic:: CentOS

    .. code-block:: shell

        $ sudo systemctl stop f5-openstack-agent.service


.. topic:: Ubuntu

    .. code-block:: shell

        $ sudo service f5-oslbaasv2-agent stop


See Also
--------

    * `F5® BIG-IP® LTM® Product Support <https://support.f5.com/kb/en-us/products/big-ip_ltm.html>`_
    * `F5® BIG-IP® User Guides <https://support.f5.com/kb/en-us/search.res.html?productList=big-ip_ltm&versionList=11-6-0&searchType=advanced&isFromGSASearch=false&query=&site=support_internal&client=support-f5-com&prodName=BIG-IP+LTM&prodVersText=11.6.0&docTypeName=Manual&q=&submit_form=&product=big-ip_ltm&pubDateFilter=all&productVersion=11-6-0&updatedDateFilter=all&documentType=manualpage&includeArchived=false>`_





