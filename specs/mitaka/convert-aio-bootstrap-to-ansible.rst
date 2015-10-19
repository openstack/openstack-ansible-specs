Convert AIO bootstrap to Ansible
################################
:date: 2015-10-16 00:00
:tags: aio, bootstrap, ansible

The process for an AIO installation of openstack-ansible involves a bash script
to do the initial bootstrapping of the AIO host. This script works well, but it
becomes difficult to update over time and a conversion to Ansible would make
future updates, such as `multi-platform-host blueprint`_, a little easier.

.. _multi-platform-host blueprint: https://blueprints.launchpad.net/openstack-ansible/+spec/multi-platform-host

Blueprint - Convert AIO bootstrap to Ansible:

* https://blueprints.launchpad.net/openstack-ansible/+spec/convert-aio-bootstrap-to-ansible

Problem description
===================

The ``bootstrap-aio.sh`` script works well, but it can be difficult to read in
a few places. Deployers who are familiar with Ansible, but not bash, may have
challenges with updating the script as well.


Proposed change
===============

At this time, the AIO installation has four steps:

* Configuration `(optional)`
* Bootstrap the AIO build
* Bootstrap Ansible
* Run the openstack-ansible playbooks

This spec proposes the following steps to replace the existing ones:

* Configuration `(optional)`
* Bootstrap Ansible
* Run AIO playbook `(if an AIO deployment is desired)`
* Run the openstack-ansible playbooks

The current AIO boostrap script is **heavily** used by various deployers as
well as other downstream projects, so changes must be made carefully.  The
proposed work for this spec would proceed as follows:

* Build out the Ansible role for bootstrapping an AIO build
* Update documentation to allow for early testing
* Change the ``bootstrap-aio.sh`` script to call the new AIO bootstrap playbook
* Update the documentation to reflect the new bootstrap script changes
* Remove the ``bootstrap-aio.sh`` script at a later date (if needed)

Alternatives
------------

The current ``bootstrap-aio.sh`` script could remain as it is now, or it could
be simplified to make it easier to read and update.

Playbook/Role impact
--------------------

The openstack-ansible playbooks themselves shouldn't change as a result of this
update.  The AIO bootstrap is a prerequisite step in the deployment right now
and that won't change after the AIO Ansible playbook is available for use.


Upgrade impact
--------------

This change would only affect greenfield deployments of AIO builds. If a
deployer has an existing AIO build deployed, they would not need to run the
AIO bootstrap playbook again, even with upgrades.

Security impact
---------------

There are no known security impacts of this change.

Performance impact
------------------

There are no known performance impacts of this change.  The Ansible AIO
playbook may be slightly slower than the bash script, but the difference should
be negligible.

End user impact
---------------

An end user would not notice this change since it would only affect deployers.


Deployer impact
---------------

If deployers are doing greenfield AIO deployments, they will need to follow new
steps and ensure they bootstrap Ansible prior to running the new AIO Ansible
playbook.  Documentation for AIO builds will require updates.

If deployers are doing deployments to multiple servers (non-AIO), their steps
for deploying openstack-ansible will not change.

Developer impact
----------------

Developers will need to make any future AIO bootstrap changes within the
Ansible playbook instead of the bash script.

Dependencies
------------

This spec doesn't depend on any other blueprint or spec at this time.

Implementation
==============

Assignee(s)
-----------

Primary assignee:

* Major Hayden (Launchpad: `rackerhacker`_, IRC: mhayden)

.. _rackerhacker: https://launchpad.net/~rackerhacker

Work items
----------

The last bulleted list in `Proposed Changes` above details out the work items.


Testing
=======

These changes will impact gating since the gating jobs run an AIO build.
However, if the bootstrap-aio.sh script is changed to call the AIO bootstrap
Ansible playbook, the gating job itself will not need to be changed.

No additional resources should be required during gating to run the Ansible AIO
playbook.

Documentation impact
====================

The documentation for AIO deployments would need to be updated with the new
steps for bootstrapping an AIO build.  The changes in the steps are in the
`Proposed Changes` section at the top of this spec.

Also, deployers would need to note which environment variables and/or Ansible
variables to set to control various parts of the deployment, such as whether or
not to deploy certain OpenStack services in their environment.

References
==========

No references at this time.

