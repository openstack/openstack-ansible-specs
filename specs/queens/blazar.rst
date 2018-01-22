Integration of Blazar with OpenStack-Ansible
##############################################
:date: 2017-12-17 00:02
:tags: openstack, blazar, opnfv, promise

Blazar is a resource reservation service for OpenStack. It is used to book
or reserve specific resources for a particular amount of time. This spec outlines
the steps required to integrate Blazar with OpenStack-Ansible.

Problem description
===================

Blazar is used to reserve OpenStack resources in advance for a specific amount of
time. However, it needs to be installed manually with OpenStack-Ansible. No role
exists to deploy it as other services are deployed.

Proposed change
===============

The change consists of creating a new role for Blazar integration with OpenStack-Ansible.
It will make it possible to deploy Blazar as part of the installation of OpenStack-Ansible,
rather then requiring to install and configure it manually.

Alternatives
------------

There are no alternatives.

Playbook/Role impact
--------------------

This is a new feature added into OpenStack-Ansible. No role currently exists. Therefore,
a new role, `openstack-ansible-os_blazar` needs to be written from scratch.

Upgrade impact
--------------

No upgrade impact.


Security impact
---------------

No security impact.

Performance impact
------------------

No performance impact.

End user impact
---------------

End user will be able to use Blazar out of the box, without going through any
manual installation and configuration. One of the endusers is Promise, an OPNFV
project, which is using Blazar, in an NFV context.

Deployer impact
---------------

No impact.

Developer impact
----------------

Little or no impact, since this feature will be optional and can be safely ignored.

Dependencies
------------

No dependencies.

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  Taseer Ahmed (Taseer)

Other contributors:
  Fatih Degirmenci (fdegir)

Work items
----------

Blazar is not available as a service for OpenStack-Ansible. No role already exists.
A new role will be developed from scratch in compliance with the standards set by the
community. The steps for developing this new role are as follows:

1. Create a new repository on GitHub.
2. Add tasks to the role.
3. Add tests for the new role.
4. Ensure that the role works well with AIO.

Testing
=======

Tests will be developed to ensure that deployment of Blazar works and also to test the
functionality of the deployed service.

Documentation impact
====================

As this would be new feature added to OpenStack-Ansible, it needs to be
documented, explaining all the configuration parameters.

References
==========

Blazar Overview

* https://wiki.openstack.org/wiki/Blazar

Blazar Installation steps

* https://docs.openstack.org/blazar/latest/install/install-without-devstack.html

OPNFV Promise

* https://wiki.opnfv.org/display/promise/Promise