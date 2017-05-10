.. _releases-and-support:

Releases and Versioning
=======================

.. seealso::

   Please see the :ref:`Partners <f5ospartners>` page for information about our OpenStack distribution platform partnerships and certifications.

Documentation
-------------

This documentation release, version |version|, corresponds to the following version(s) of each F5 OpenStack Integration component.

.. table:: Docs and Component Versioning
   :header-alignment: center center
   :column-alignment: left right

   ================================ ===========================
   Component                        Version
   ================================ ===========================
   f5-openstack-agent               |agent-versions|
   -------------------------------- ---------------------------
   f5-openstack-heat                |heat-versions|
   -------------------------------- ---------------------------
   f5-openstack-heat-plugins        |plugins-versions|
   -------------------------------- ---------------------------
   f5-openstack-lbaasv2-driver      |driver-versions|
   -------------------------------- ---------------------------
   f5-openstack-lbaasv1 [#eosd]_    7.x, 8.x, 9.x
   ================================ ===========================

\

.. [#eosd] F5 ceased to repair defects and perform maintenance on the F5 OpenStack LBaaS version 1 integration as of the Openstack Ocata release (April 2017). See the :ref:`Neutron LBaaSv1 section <neutronlbaasv1support>` below for more information.

F5 OpenStack Heat Integration
-----------------------------

.. table:: OpenStack & F5 BIG-IP compatibility
   :header-alignment: center center center center
   :column-alignment: right right right right

   ==================== =============== =================== ===================
   F5 Heat templates    F5 Heat plugins OpenStack version   BIG-IP version(s)
   ==================== =============== =================== ===================
   7.x                  7.x             Kilo                11.5.2+

                                                            11.6.x

                                                            12.0.x
   -------------------- --------------- ------------------- -------------------
   8.x                  8.x             Liberty             11.5.2+

                                                            11.6.x

                                                            12.0.x

                                                            12.1.x
   -------------------- --------------- ------------------- -------------------
   9.x                  9.x             Mitaka              11.5.2+

                                                            11.6.x

                                                            12.0.x

                                                            12.1.x
   -------------------- --------------- ------------------- -------------------
   **COMING SOON**                      Newton              11.5.2+

                                                            11.6.x

                                                            12.0.x

                                                            12.1.x
   ==================== =============== =================== ===================


F5 OpenStack LBaaS Connector
----------------------------

Neutron LBaaSv2
```````````````

.. table:: OpenStack LBaaSv2 & F5 BIG-IP compatibility
   :header-alignment: center center center
   :column-alignment: right right right

   ================================ =================== ===================
   F5 LBaaS Connector version(s)    OpenStack version   BIG-IP version(s)
   ================================ =================== ===================
   8.x                              Liberty             11.5.2+

                                                        11.6.x

                                                        12.0.x

                                                        12.1.x
   -------------------------------- ------------------- -------------------
   9.x                              Mitaka              11.5.2+

                                                        11.6.x

                                                        12.0.x

                                                        12.1.x
   -------------------------------- ------------------- -------------------
   **COMING SOON**                  Newton              11.5.2+

                                                        11.6.x

                                                        12.0.x

                                                        12.1.x
   ================================ =================== ===================

.. table:: Linux OS Compatibility
   :header-alignment: center center center
   :column-alignment: right right right

   ================================ =============== ====================
   F5 LBaaS Connector version(s)    RHEL version(s) Ubuntu version(s)
   ================================ =============== ====================
   8.x, 9.x                         6, 7            12, 14
   ================================ =============== ====================

.. _neutronlbaasv1support:

Neutron LBaaSv1
```````````````

.. important::

   **End of Software Development for F5 OpenStack LBaaS version 1**

   F5 announced the End of Software Development (EoSD) for the F5 OpenStack LBaaS version 1 integration effective October 1, 2016.
   This announcement is in compliance with the OpenStack community deprecation of the OpenStack Neutron LBaaS version 1 plugin.
   Customers are encouraged to move to OpenStack LBaaS version 2.

   F5 ceased to repair defects and perform maintenance on the F5 OpenStack LBaaS version 1 integration as of the OpenStack Ocata release (April 2017).

   *The table below is provided for informational purposes only.*

\
   .. table:: OpenStack LBaaSv1 & F5 BIG-IP compatibility
      :header-alignment: center center center
      :column-alignment: right right right

      ================================ =================== ========================
      F5 LBaaSv1 Connector version(s)  OpenStack version   BIG-IP version(s)
      ================================ =================== ========================
      7.x                              Kilo                11.5.2+, 11.6.x, 12.0.x
      -------------------------------- ------------------- ------------------------
      8.x                              Liberty             11.5.2+, 11.6.x, 12.0.x
      -------------------------------- ------------------- ------------------------
      9.x                              Mitaka              11.5.2+, 11.6.x, 12.0.x
      ================================ =================== ========================

