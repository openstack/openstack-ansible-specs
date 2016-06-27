Only support venv installs
##########################
:date: 2016-06-27 13:30
:tags: python, venv, deployment

The purpose of this spec is remove support for installing OpenStack services
and dependent pip packages outside of Python virtual environments.

  * https://blueprints.launchpad.net/openstack-ansible/+spec/only-install-venvs

Problem description
===================

Conflicts between system packages and globally installed Python pip packages
can lead to broken services and strange behavior. The default installation
option of OpenStack services since the Liberty release has been to use virtual
environments to isolate each individual service. This should be the only
supported option going forward.


Proposed change
===============

Each role will be updated to remove tasks and variables related to allowing the
option of installing pip packages outside of a virtual environment. The tasks
which currently handle installing virtual environments will also be updated to
ensure that they are idempotent and can recover properly from an interruption
in a previous run of the same role.


Alternatives
------------

Leave the roles as they are. Deployment of OpenStack services would continue
being supported through either virtual environments or installed as global
system Python packages.


Playbook/Role impact
--------------------

See `Proposed change`_.


Upgrade impact
--------------

Installing services to virtual environments has been the default since the
Liberty release. If any Mitaka deployments are still configured to not install
services to virtual environments, they will be forced to beginning in the
Newton release.


Security impact
---------------

N/A.


Performance impact
------------------

Tasks which are currently being skipped will be removed, which could slightly
decrease role run times.


End user impact
---------------

N/A.


Deployer impact
---------------

The *_venv_enabled variables will no longer exist and will have no effect if
set by a deployer.


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
  https://launchpad.net/~jimmy-mccrory (jmccrory)


Work items
----------

* Remove tasks related to installation of pip packages outside of a venv from
  each role
* Remove variables which currently toggle installation of pip packages to a
  venv from each role
* Update each role to make tasks which create and install packages to a venv
  more resilient and idempotent


Testing
=======

Both integrated and independent role gate testing are already only installing
services to virtual environments.


Documentation impact
====================

Should be minimal.


References
==========

N/A.
