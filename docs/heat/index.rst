.. _heat-home:

F5 OpenStack Heat Integration
=============================

F5's `OpenStack Heat`_ Integration consists of the `F5 OpenStack Heat templates`_ and `F5 OpenStack Heat plugins`_.


General Prerequisites
---------------------

The following prerequisites apply to most of the F5 OpenStack Heat integration's documentation.

Requirements
````````````

- Understanding of `BIG-IP Local Traffic Management Basics`_.
- BIG-IP `Local Traffic Manager`_ (LTM) `Virtual Edition`_ (VE) image file in qcow.zip format (any size). [#]_
- An operational |os-deployment| with |neutron| and |heat| installed. [#]_

Caveats
```````

- You must host your BIG-IP VE image in a location accessible via 'http'.
  The F5 Heat templates can not retrieve files via 'https'.
  File uploads via OpenStack Horizon are not supported.
- BIG-IP VE images come in `different sizes`_.
  Choose the correct :ref:`F5 flavor <big-ip_flavors>` for your image.

Heat Plugins
------------

The :ref:`F5 Heat plugins <heatplugins:home>` enable BIG-IP objects for use in OpenStack.
The Heat plugins use the :ref:`F5 python sdk <f5sdk:F5 Python SDK Documentation>` to communicate with BIG-IP via the iControl REST API.


Templates
---------

The F5 Heat templates provision resources and BIG-IP services in an OpenStack cloud.
F5's Heat templates follow the `OpenStack Heat Orchestration Template`_ (HOT) specification. They can be used in conjunction with `F5 iApps <https://devcentral.f5.com/wiki/iApp.HomePage.ashx>`_ to deploy BIG-IP VE instances and Local Traffic Manager services.

The F5 Heat templates come in two flavors: :ref:`supported <heat:f5-supported_home>` and :ref:`unsupported <heat:unsupported_home>`.
The `F5 OpenStack Heat templates`_ product documentation contains download links for all supported templates.

.. warning::

   The :ref:`unsupported <heat:unsupported_home>` templates provided in the `f5-openstack-heat`_ GitHub repo are considered 'use-at-your-own-risk'.
   Templates in the unsupported directory are not supported by F5, regardless of your account's support agreement status.


.. seealso::

   * :fonticon:`fa fa-github` `f5-openstack-heat`_
   * :fonticon:`fa fa-github` `f5-openstack-heat-plugins`_
   * :fonticon:`fa fa-book` `F5 OpenStack Heat templates`_
   * :fonticon:`fa fa-book` `F5 OpenStack Heat plugins`_


.. [#] `How to Buy <https://f5.com/products/how-to-buy>`_
.. [#] Unsure how to get started with OpenStack? Consult one of F5's :ref:`OpenStack Platform Partners <f5ospartners>`.

.. _BIG-IP Local Traffic Management Basics: https://support.f5.com/kb/en-us/products/big-ip_ltm/manuals/product/ltm-basics-13-0-0.html
.. _Local Traffic Manager: https://f5.com/products/big-ip/local-traffic-manager-ltm
.. _Virtual Edition: https://f5.com/products/deployment-methods/virtual-editions
.. _different sizes: https://support.f5.com/csp/article/K14946
.. _OpenStack Heat Orchestration Template: https://docs.openstack.org/developer/heat/template_guide/hot_spec.html
