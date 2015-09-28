Allow os_* services to use a venv
#################################
:date: 2015-05-08 00:00
:tags: python, venv, deployment

Enable the ability for a role to deploy OpenStack python code inside a venv

Blueprint:
  https://blueprints.launchpad.net/openstack-ansible/+spec/enable-venv-support-within-the-roles


Problem description
===================

There are two problems that we need to start anticipating:
  * Some OpenStack services are running on physical hosts in the root
    namespace. This creates a situation where it's possible for a service to
    have conflicting requirements with what is already on the host installed
    through the host package manager. In these situation we've found some
    instabilities that needed workarounds to ensure there are no stability or
    usage issues with the service.
  * OpenStack services have started moving toward a non-integrated release
    which will allow projects to change their release cycle / cadence which
    will effect versions of services that we deploy. Additionally, these
    projects may choose to use dependencies outside of what is set in Global
    requirements.

The use of on metal services, the change in release cycles / cadence, and
the likelihood of projects using requirements that conflict with one another
requires more separation between the installed projects which lends itself
to using a virtual environment for installed OpenStack Python code.


Proposed change
===============

* Each os_* role will be modified to support a service running in a virtual
  environment. This will mean a few new variables to the defaults per-role to
  determine where the venv will live, change in pip package requirements as
  the virtualenv package will need to be installed first, changes to the init
  scripts to support a virtual environment, and a change to the sudoers file
  to allow the virtual environment bin path to be saved when executing a
  rootwrap command.

* The roles will support the option to deploy in a venv or not. This will be
  disabled ``false`` by default.

* The playbooks will have an option within them to enable or disable venv
  support at run time.

* Each venv will be named and tagged such that its unique as it pertains to
  the deployment. This will allow for package upgrade and downgrades on long
  lived deployements to take place without manual intervention or messing
  with the hosts packages which may have been installed as part of the base
  kick and using the operating system package manager.


Alternatives
------------

Leave things unchanged or further pursue re-containerizing services that have
been moved to the host. If we decide to go the route of re-containerizing
projects that have been moved to the host's namespace we will need to invest
in kernel development to fix several issues we encountered which forced the
move to running "is_metal" in the first place.


Playbook/Role impact
--------------------

See `Proposed change`_.


Upgrade impact
--------------

The use of venvs within an environment will not effect an existing
deployment nor have any adverse effects on upgrades. Upgrading a
service that hadn't used venvs in the past will be taken care of
automatically as init scripts, sudoers files, and rootwrap configs
will be changed to support the new venv install.

The benefit of running a service in a venv is apparent when dealing
with downgrading a package requirement. This issue has been seen a
few times where an upstream OpenStack project has downgraded a
python package requirement in the middle of a release. In the
current deployment system an administrator is required to manually
intervene to resolve package downgrade issues. If the system was using
a venv and was tagged based on a given deployment upgrading from
one to release to another is as simple as re-running the role from the
new released version. The result will be a new venv created for the
service and the version. This has an upgrade side effect that will
allow for Kilo to Liberty upgrades without having to deal with
a epoch wheel build or munging of the wheels repo further
simplifying an upgrade in terms of what will be required by the end
user.


Security impact
---------------

N/A.


Performance impact
------------------

While not directly related to the implementation of this spec it would
be possible for us to extend the virtualenv implementation to allow for
building and redistribution of pre-built virtualenvs as a means of
speeding up and maintaining reproducibility within an environment.


End user impact
---------------

N/A.


Deployer impact
---------------

When working within a container access to the service management utilities
(nova-manage, cinder-manager, etc...) the deployer or administrator working
on an environment will need to sourced/activated the virtualenv before
running the tools. While this is an extra step there are no other changes
that will need to be addressed in the typical deployer workflow.


Developer impact
----------------

N/A.


Dependencies
------------

N/A.


Implementation
==============

Assignee(s)
-----------

Primary assignee:
  <cloudnull> - https://launchpad.net/~kevin-carter

Secondary assignee:
  Anyone who wants to help


Work items
----------

* Update all roles to support venvs
* Add a variable to the OSA playbooks to enable venv support within the roles.

Testing
=======

* Testing this will rely on the gate as a convergence test.
* This is implemented in Liberty we can create a simple periodic job in
  OpenStack infra to test upgrades. The upgrade testing will report back
  to the OpenStack QA mailing list and key of their periodic job queue.


Documentation impact
====================

* Documentation will need to be written to acknowledge the venv based
  deployment and how deployers are to interact with the management tools as
  provided by the service.


References
==========

Related Bug:
  * https://bugs.launchpad.net/openstack-ansible/+bug/1488315

