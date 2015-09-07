Remove upstream repo dependency
###############################
:date: 2015-07-19
:tags: repo, repo-servers, repo-clone, pip-wheel

The purpose of this spec is to remove the repo-clone play from OSAD.

* https://blueprints.launchpad.net/openstack-ansible/+spec/Remove-upstream-repo-dependency


Problem description
===================

Presently the repo-clone-mirror play is responsible for cloning the upstream
repository that Rackspace maintains into the repo containers. While this process
is simple enough it does bring with it a reliance on an upstream deployer/vendor.
OSAD already has the ability to build its own python packages which is the process
used to do all gate check testing so it should also be the default means to deploy
an OSAD environment.


Proposed change
===============

* Remove the repo-clone-mirror.yml play
* Change repo-install.yml to use repo-build.yml as it's included method.
* Modify the pip install role to remove the install requirement using the upstream
  mirror.


Alternatives
------------

Leave everything the way it is.


Playbook impact
---------------

Changes the repo create process to always build. This will only impact deployers
that are using the repo-servers and will ensure that the system is always building
the correct packages.

When bootstrapping a new environment the pip install role is used throughout the stack.
This would modify that role to always pull upstream pip unless otherwise instructed,
through the use of *user_vars*, to go elsewhere.


Upgrade impact
--------------

n/a


Security impact
---------------

n/a


Performance impact
------------------

Repo clone was intended to be a faster means of delivering packages to the deployment
infrastructure however in testing repo clone and repo build operate at roughly the
same speed.


End user impact
---------------

n/a


Deployer impact
---------------

This change will be unnoticeable to the deployer.


Developer impact
----------------

n/a


Dependencies
------------

n/a

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  https://launchpad.net/~kevin-carter ``cloudnull``


Work items
----------

* Delete the repo-clone-mirror.yml play
* Change the include in repo-install.yml 's/repo-clone-mirror.yml/repo-build.yml/'


Testing
=======

This is already being tested on every build within upstream OSAD.


Documentation impact
====================

n/a


References
==========

n/a
