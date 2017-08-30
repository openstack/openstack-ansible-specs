Integration of Congress with OpenStack Ansible
##############################################
:date: 2017-08-30 00:02
:tags: openstack, congress

Blueprint on Launchpad:
  * https://blueprints.launchpad.net/openstack-ansible/+spec/role-congress

Congress is the policy framework for OpenStack. This spec introduces the work required
to deploy Congress, as a service for OpenStack Ansible.

Problem description
===================

There are many policy frameworks for OpenStack. However, very few of them
come with OpenStack Ansible. They need to be manually configured and installed.
The aim of this spec is to deploy Congress with OpenStack Ansible, provided as a
service to OpenStack Ansible and OpenStack users in general.

Proposed change
===============

The change consists of integrating Congress with OpenStack Ansible during deployment
phase of OpenStack.

Alternatives
------------

Many policy frameworks for OpenStack exist. Tacker is one of them and has already been
integrated with OpenStack Ansible. However, Tacker is more of a VNF Manager, mostly used
for NFV related activites such as Service Function Chaining etc.

Playbook/Role impact
--------------------

This is a new feature being introduced.An existing role does not already exist.
A new role will be developed, e.g `openstack-ansible-os_congress`. This new role
will be developed as per the steps outlined by the community.

Upgrade impact
--------------

No upgrade impact since this would be the first time implementation of the proposed
change.

Security impact
---------------

No security impact.

Performance impact
------------------

Performance impact should be very low, it only needs a few preliminary packages.

End user impact
---------------

Congress uses a simple declarative language to define real world policies. Currently
it needs to be manually configured and deployed. This feature would enable the users to
use Congress as a service, and be able to manage OpenStack more efficiently.

Deployer impact
---------------

No default policies will be enforced. If the deployer chooses to enable Congress service,
policies need to be defined as per the requirements.

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

Congress is not available as a service for OpenStack Ansible. No role already exists.
A new role will be developed from scratch in compliance with the standards set by the
community. The steps for developing this new role are as follows:

1. Create a new repository on GitHub.
2. Add tasks to the role.
3. Add tests for the new role.
4. Ensure that the role works well with AIO.

Testing
=======

Tests will be developed to ensure that deployment of Congress works and also test the
functionality of the deployed service.

Documentation impact
====================

As this would be new feature added to OpenStack Ansible, it needs to be
documented, explaining all the configuration parameters.

References
==========

Congress Overview

* https://wiki.openstack.org/wiki/Congress

Congress Installation steps

* https://docs.openstack.org/congress/latest/install/index.html#separate-install