.. _big-ip_flavors:

Nova Flavors for BIG-IP Virtual Edition images
----------------------------------------------

BIG-IP Virtual Edition (VE) images come in different sizes -- LTM_1SLOT, LTM, and ALL -- each of which requires a minimum of 2 CPUs and 4GB RAM.

The following F5 Knowledge Base articles provide information regarding image sizes and hardware requirements:

- `K14946: Overview of BIG-IP VE image sizes <https://support.f5.com/csp/article/K14946>`_
- `K15796: Hardware requirements to host BIG-IP VE on private cloud platforms <https://support.f5.com/csp/article/K15796>`_

In OpenStack Mitaka and earlier, you can use the following default Nova flavors for BIG-IP VE images. [#newton]_

- m1.medium
- m1.large
- m1.xlarge

See the OpenStack `Manage Flavors <https://docs.openstack.org/horizon/latest/admin/manage-flavors.html>` doc for the specifics of each default flavor.

.. seealso::

   * :ref:`How to add the F5 Nova flavors to OpenStack <add-nova-flavors>`
   * `OpenStack Admin Guide: Manage flavors <https://docs.openstack.org/horizon/latest/admin/manage-flavors.html>`_

.. rubric:: Footnotes
.. [#newton] OpenStack Newton doesn't have any default Nova flavors.

.. _BIG-IP VE Linux KVM Setup Guide: https://support.f5.com/kb/en-us/products/big-ip_ltm/manuals/product/bigip-ve-setup-linux-kvm-13-0-0/2.html
.. _Nova flavors: https://docs.openstack.org/horizon/latest/admin/manage-flavors.html
