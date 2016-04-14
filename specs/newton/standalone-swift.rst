standalone-swift
#################################
:date: 2015-07-07 22:00
:tags: swift, aio, tests

This spec exists to allow for testing a diferent deployment methodogy, namely
swift deployments.  The problem is the openstack_user_config.yml.aio file
defines hosts that are not needed for an AIO deployment.

* https://blueprints.launchpad.net/openstack-ansible/+spec/standalone-swift


Problem description
===================

Deploying aio for testing deploys all Openstack services only swift is desired.
We are not testing this deployment type.


Proposed change
===============

* add openstack_user_config.yml.aio.swift for swift only deployments.

* add/modify the deployment scripts to add a switch for swift only deployments.

* modify tests to allow for swift only deployments.


Alternatives
------------

N/A


Playbook impact
---------------

Minimal to no impact to the actual playbooks.


Upgrade impact
--------------

N/A


Security impact
---------------

N/A


Performance impact
------------------

N/A


End user impact
---------------

Allows the end user to use the openstack_user_config.yml.aio.swift file as a
template to base their own swift deployments.


Deployer impact
---------------

The playbooks would remain unchanged, only deployers using the scripts may
need to change, this does not alter default behavior.


Developer impact
----------------

This would allow testing of standalone swift deployments.


Dependencies
------------

N/A


Implementation
==============

Assignee(s)
-----------

Primary assignee:
  prometheanfire


Work items
----------

* create aio file

* add/alter scripts to allow for standalone swift testing (tempest changes)

* add test to project_config

* enable test in openstack-ansible


Testing
=======

This will add a test/vote to openstack-ansible


Documentation impact
====================

Possibly pointing out the openstack_user_config.yml.aio.swift file as a
template for larger deployments and documenting the new environment variables.


References
==========

N/A

