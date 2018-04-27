Install OpenStack services from distribution packages
#######################################################
:date: 2018-03-27 00:00
:tags: roles, deployment

Blueprint on Launchpad

  * https://blueprints.launchpad.net/openstack-ansible/+spec/openstack-distribution-packages

This spec outlines the work required to enable the OpenStack-Ansible roles to
install the OpenStack services using the distribution packages from the distribution
Cloud repositories.

Problem description
===================

OpenStack-Ansible installs the OpenStack services from the source. Whilst this
is great in terms of flexibility, it creates some problems such as:

* Long deployment times since wheel packages need to be built and distributed.
* Unsupported installations by distributions. The versions of OpenStack services
  built from source do not necessarily match what distributions test together as
  part of their integration and verification process so it's hard for them to
  provide support for such installations. As a result of which, operators have
  limited options when seeking technical support for their deployments.

Proposed change
===============

Add an additional installation method to all the OpenStack-Ansible roles in
which the services will be installed using the packages provided by the
distributions themselves. The default installation method will not change.

Alternatives
------------

N/A

Playbook/Role impact
--------------------

All the OpenStack Ansible roles which install OpenStack services (os_*) will be
impacted by the proposed change. A new variable will be made available on per-role
basis to allow deployers to select the preferred installation method.

Switching from one installation method to the other will not be supported.
This can be clarified on the Deployer's documentation and also explicitly detected
and prevented in the Ansible playbooks possibly by storing a local fact on the host to
denote the installation method and checking it during upgrades.

Upgrade impact
--------------

Upgrades should not be impacted since the default installation method will not
change.

Security impact
---------------

The security of the overall installation will not change since distributions
normally backport security fixes which are already present in the upstream packages
so both installations methods will offer the same level of security reassurances.

Performance impact
------------------

The overall performance of the deployment will likely be improved since the
distribution packages normally have their default settings tweaked and
optimized to match each distribution's environment and needs.

End user impact
---------------

N/A

Deployer impact
---------------

The benefit of this new method for deployer's is twofold:

- Use supported packages by distributions and provide feedback back to them.
  This benefits both distributions and operators since both ends use packages
  which have passed integration and functional testing before being released.
- Shorten deployment times since distribution packages are used instead of building
  new ones from source.

Developer impact
----------------

N/A

Dependencies
------------

N/A

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  Markos Chandras (hwoarang)

Work items
----------

The following work items are the same across all impacted roles

* Move existing installation tasks to a new file (``${role}_install_source.yml``)
* Create new file (``${role}_install_distro.yml``) with a set of tasks for distribution
  installations if necessary.
* Add new variable to allow deployers to select installation method (``${role}_install_method``)
* Dynamically include the appropriate installation file based on the variable's value

Testing
=======

Since the default installation method does not change, no new tests are required.
However, developers may choose to add new jobs on per distribution basis to test
the new installation method.

Documentation impact
====================

Documentation needs to be modified to explain how to use the ``distribution``
installation method.

References
==========

N/A
