Create a tls load balancer
==========================

The example command below shows how to create a listener that uses the ``TERMINATED_HTTPS`` protocol. You'll need to specify the protocol (``TERMINATED_HTTPS``); port; and the location of the `Barbican container <http://docs.openstack.org/developer/barbican/api/quickstart/containers.html>`_ where the certificate is stored.

.. code-block:: shell

    $ neutron lbaas-listener-create --name listener2 --protocol TERMINATED_HTTPS --protocol-port 8443 --loadbalancer lb1 --default-tls-container-ref  http://localhost:9311/v1/containers/db50dbb3-70c2-44ea-844c-202e06203488


.. important::

    You must configure Barbican, Keystone, Neutron, and the F5 agent before you can create a tls load balancer.

    See the `OpenStack LBaaS documentation <https://wiki.openstack.org/wiki/Network/LBaaS/docs/how-to-create-tls-loadbalancer>`_ for further information and configuration instructions for the OpenStack pieces.

    The necessary F5 agent configurations are described in :ref:`Certificate Manager / SSL Offloading`.
