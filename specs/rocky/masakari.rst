Integration of Masakari with OpenStack-Ansible
#################################
:date: 2018-03-22 14:00
:tags: openstack, masakari, masakari-monitors

Blueprint on Launchpad:
  * https://blueprints.launchpad.net/openstack-ansible/+spec/masakari-ansible-plugin

Masakari provides Virtual Machine High Availability (VMHA) service for OpenStack clouds
by automatically recovering the KVM-based Virtual Machine(VM)s from failure events such
as VM process down, provisioning process down, and nova-compute host failure. It also
provides API service for managing and controlling the automated rescue mechanism.
The Masakari service consists of the following components:

* masakari-api:
  An OpenStack-native REST API that processes API requests by sending
  them to the ``masakari-engine`` over `Remote Procedure Call (RPC)`.

* masakari-engine:
  Processes the notifications received from ``masakari-api`` by execcuting the
  recovery workflow in asynchronus way.

* masakari-monitors:
  Monitors for Masakari provides Virtual Machine High Availability (VMHA) service for OpenStack
  clouds by automatically detecting the failure events such as VM process down, provisioning
  process down, and nova-compute host failure. If it detects the events, it sends notifications
  to the masakari-api.

This spec outlines the steps required to integrate Masakari with OpenStack-Ansible.

Problem description
===================

Masakari provides Instances High Availability Service for OpenStack clouds
by automatically recovering failed Instances. However, it needs to be installed
manually with OpenStack-Ansible. No role exists to deploy it as other services are deployed.

Proposed change
===============

The proposed changes would include:

* Import a proof of concept role for Masakari from
  https://github.com/NirajSingh90/openstack-ansible-os_masakari to
  ``openstack-ansible-os_masakari``
* Follow the usual path described in the developer documentation.

Alternatives
------------

There are no alternatives.

Playbook/Role impact
--------------------

This is a new feature added into OpenStack-Ansible. No role currently exists. Therefore,
new role, `openstack-ansible-os_masakari` needs to be written from scratch.

Upgrade impact
--------------

No upgrade impact since this would be the first implementation of the proposed change.

Security impact
---------------

No security impact.

Performance impact
------------------

No performance impact.

End user impact
---------------

End user will be able to use masakari as a service within OpenStack-Ansible.

Deployer impact
---------------

Deployers will need to enable Masakari deployments if they choose to use this.
Masakari will not be deployed by default.

Developer impact
----------------

No impact.

Dependencies
------------

By employing a combination of Corosync and Pacemaker, OpenStack Masakari creates a
cluster of servers, detecting and reporting failure of hosts in the cluster.
So masakari is dependent on Corosync and Pacemaker.

We will reuse an external role for corosync and pacemaker to not re-invent the wheel,
like the one found in https://github.com/leucos/ansible-pacemaker-corosync .

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  Niraj Singh (IRC: niraj_singh)

Work items
----------

Masakari is not available as a service for OpenStack-Ansible. No role already exists.
A new role will be developed from scratch in compliance with the standards set by the
community. It will be added under https://github.com/openstack/openstack-ansible-os_masakari

Note: Masakari role will install below services:
maskari-api
masakari-engine
masakari-processmonitor
masakari-hostmonitor
masakari-instancemonitor

masakari-processmonitor, masakari-hostmonitor and masakari-instancemonitor will be
installed only on nova-compute nodes

Testing
=======

Tests will be developed to ensure that deployment of Masakari works. Masakari
doesn't have tempest tests therefore we will start by testing the API responses
codes. Masakari-monitor and Masakari-engine services tests will be added in
future using third party CI tests.

Documentation impact
====================

As this would be new feature added to OpenStack-Ansible, it needs to be
documented, explaining all the configuration parameters.

References
==========

Masakari Overview

* https://wiki.openstack.org/wiki/Masakari

Masakari developer/operator documentation

* https://docs.openstack.org/masakari/latest
