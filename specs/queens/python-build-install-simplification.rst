Python Build/Install Process Simplification
###########################################
:date: 2017-09-06 13:00
:tags: python, build, source, repo

The current python wheel/venv build process is not easily understood, and the
install process has become complicated. This blueprint aims to work towards
making it simpler to deploy, simpler to understand and to make many of the
current features which are forced on all deployers to be opt-in.

Launchpad Blueprint:
https://blueprints.launchpad.net/openstack-ansible/+spec/python-build-install-simplification

Problem description
===================

Building
~~~~~~~~
The Python repository used for OpenStack-Ansible deployments is used to
prepare `Python wheels`_ for any git- or pypi-sourced packages for an
environment. Using wheels speeds up the installation of the package and
takes away the need to install the distribution packages required to
compile the package when installing.

The repository preparation process also prepares `Python virtualenvs`_
for all OSA roles with the prefix ``os_`` (which are expected to be
OpenStack services) in order to speed up the deployment of the services
by downloading a complete virtualenv instead of installing the packages
individually for every host that needs the service.

The ``py_pkgs`` lookup, which pulls together the information used by the
build process. It is a black box in terms of what it does, making some
decisions about the information it reads and outputs which are not
documented anywhere other than in the code itself. The code is not easily
modified without breaking the process and is therefore most often left
alone and not well maintained, resulting in an increasing amount of technical
debt. The subsequent jinja in the repo-build role which processes it is
tough to work through and not easily maintained. While both of these could
be adjusted to make use of different plugins or filters, it would remain
a set of black boxes which are complex to untangle.

The way that git repositories are specified and parameters are provided
to the build process does not scale very well. Each git repo requires at
least two flat variables to be set (``git_repo`` and ``git_install_branch``)
and can optionally have more set. This model of setting variables makes it
really easy to override individual settings, but requires the use of a
pattern match mechanism to discover all the settings (which is why we use
the lookup to do it). The settings are also put in disparate places, making
them hard to find - defaults/repo_packages, role/defaults. It is not very
obvious to most newcomers how to change them and it is not obvious to many
veterans what many of the settings mean. It often requires a lot of code
walking to understand the meaning of some settings like ``venvwithindex``
and ``ignorerequirements``.

The git clone process used to fetch the git sources in order to use when
building is done asynchronously in order to improve the time to completion,
however individual asynchronous tasks cannot be retried in Ansible, and the
git clones commonly fail. This is an Ansible limitation which we could work
around by implementing our own action module, but this would increase the
technical debt as we would have to constantly keep the module code updated
as we update to later versions of Ansible.

When building wheels, pip has no way of resolving all dependencies up-front.
The only capability it has is to resolve the requirements for the current
package. It then processes each package requirement in turn. To do so requires
downloading the package and unpacking it to read the requirements. This is a
sequential process and therefore takes a long time when processing packages
with a lot of requirements as is typical for OpenStack projects.

In Kilo the OpenStack requirements management process did not have the jobs
which tested the co-installability of all OpenStack packages and produced the
``upper-constraints.txt`` file as a manifest of which package versions worked
together. We therefore needed to do our own processing of all python packages
which would be installed by the roles and had to compile a set of requirements
and constraints across them all for the purpose of building the wheels, and
ensuring that the installed set were consistent for a build. When the
OpenStack requirements repository started publishing the upper-constraints
file we adopted it immediately to help keep builds more consistent. However,
we still produce our own ``requirements_absolute_requirements.txt`` file
which is used for all pip install tasks in order to ensure consistency and
to ensure that the packages we built from git are used (instead of making
the install in the role do the install from the git source, it installs from
the wheel held in the repo server). However this is not practical any more
as there are requirements for different services and needs which are not
resolvable down to a common set - we need to be able to allow the installation
of any version of packages and only apply constraints when needed.

Some of the venvs we build do not adhere to the OpenStack requirements process
and therefore sometimes cannot be built using the upper constraints file.
There has also been some interest in being able to do mixed series deployments
instead of homogenous deployments. This would involve preparing a venv
containing packages from a different series with a different set of
constraints. Currently the constraints used in the repo build process are
global - we only have the ability to enable/disable their use when building
venvs. It would be better to be able to specify a global fallback for
constraints, but to allow per venv constraints too.

The use of Python 2.7 for OpenStack and Ansible is waning and the need to
shift everything to use Python 3.5 has arisen as a new requirement. The
tooling will need to be shifted to implement the venvs using Python 3.5
where applicable, but may still need to prepare venvs using Python 2.7 if
a service does not yet support running in a Python 3.5 environment.

In Newton we introduced the ability to do multi-architecture builds to cater
for multiple architectures, then had to also split out multi-distro builds due
do wheel/venvs references to C libraries being different for each distro due
to the libraries available being different. Currently this is working, but it
makes the repo build process much more complex and take a lot more time. The
process to synchronise the per-distro and per-architecture built artifacts is
error prone and confuses many newcomers to the project.

In order to facilitate using the repo-server to respond to pip index queries,
multiple directories and symlinks have been used to prepare the appropriate
structure so that the correct responses are given back to pip. The process of
setting up all the symlinks is very time consuming and in some places the
process may cause dead links, especially when rebuilding for a specific release
tag.

Storing
~~~~~~~
Once the wheels, venvs and other artifacts are built for an environment they
are stored and synchronised between the repo containers using a combination of
rsync and lsyncd. While this sync process is generally OK, it is commonly a
cause for confusion and requires a complex troubleshooting process to figure
out why packages are not present.

Installing
~~~~~~~~~~
The consumption of the prepared wheels and virtualenvs has changed over
time. With the introduction of ``developer_mode`` into the roles there
is a lot of code and functionality duplication between the repo build
process and the role installation process.

The need to cater for the optional inclusion of a variety of plugins/drivers
in the venvs either through the use of additional Python packages or by
symlinking system packages into the venv (when the package is proprietary or
unavailable via git or pypi) causes further complexity in the process.

When executing a pip installation, pip always looks for packages in the
following order: local cache, local folder, default index, extra indexes.
Pip will always check all locations before deciding which to use for the
installation. This means that if there are multiple indexes used, it queries
them all. This can be slow if any of those are not local to the environment.

.. _Python wheels: http://pythonwheels.com
.. _Python virtualenvs: https://virtualenv.pypa.io

Proposed change
===============

* Change the repo build process to, by default, only build wheels for git
  sources it is given without also building the dependencies. The ability
  to build all wheels will still be there, but will not be the default
  behaviour. This will cut down the time taken in this process when in CI,
  development environments or small online environments where it is not
  necessary to build/store all the wheels. The full build will only be
  necessary for offline deployments and for environments where the deployer
  specifically opts-in to ensuring that everything is built.

* Replace the current storage structure for wheels with a flat directory.
  This directory will be served via the pypi API provided by the very simple
  `pypiserver`_ application. If we need to continue to provide per-distro
  or per-architecture wheels then we could implement distro/arch indexes
  which are supplied by individual folders. However, it is unlikely that
  this will be necessary.

* Use nginx as a reverse proxy which responds to requests from pip by first
  trying against the local pypiserver, then against tarballs.openstack.org
  and then against pypi. This will allow nginx to cache all downloaded
  packages (speeding up subsequent requests) without the repo server having
  build them.

* Implement changes to the roles to allow service-specific constraints to
  be applied when building venvs. This allows a CI process to build service
  venvs and to publish the list of tested versions for that service. Then
  for production builds the published list can be used as a constraint for
  the venv to ensure that production builds use the same versions. This
  solves a problem we have today where some projects (eg: tempest, rally,
  gnocchi) have to be built unconstrained as they do not conform to the
  global requirements process.

* Implement changes to each role to handle the wheel building and venv
  building, but do it in such a way that only the build can be executed
  by using tags, setting a specific flag, or include_role and tasks_from.
  The specific dependencies can then be itemised in the role and the role
  can be used for artifact preparation.

* Remove all pip install activities from hosts, replacing them with the
  use of distro packages exclusively for any python requirements on the
  hosts. We should avoid implementing as many python packages on the host
  as possible and focus all efforts on implementing everything we need
  (including the Ansible requirements for targeted hosts) into venvs.
  All Ansible tasks should then specifically use the appropriate venv
  when executing tasks, avoiding the use of any python libraries on the
  host. This prevents system package conflicts and will reduce the host
  package installation requirements.

* Implement a playbook which is optionally used to prepare pre-built venvs
  for an environment as they are today. If a deployer wishes to prepare
  the venvs in a build process, the playbook should be exercised in the
  build process and should be executed on a designated 'build host' which
  will make use of ephemeral containers and/or virtual machines on the build
  host to exercise the builds for the necessary distribution and architecture
  combinations.

* Remove the complex git caching/staging process which exists today and
  make the use of the repo server for git caching for the services that
  require it (eg: nova-console uses novnc/spice from git) entirely optional.

* Implement a playbook which can be used to stage offline installs by
  downloading all built artifacts (completed, perhaps by a CI job) to the
  deployment host, then distributing them appropriately.

* Simplify the constraints management by implementing the use of
  --constraints in the following order:

  --constraint user-specified-constraints.txt
  --constraint openstack-ansible-pins.txt
  --constraint openstack-upper-constraints.txt

  This would replace the current method which merges the various constraints
  into one file, requiring a fair amount of jinja magic because a single
  file cannot have two constraints and resolve successfully into a single
  result as we need in our current mechanism.

* Implement changes to roles to ensure that the build process and the
  packages only required when building (dev headers, etc) are only
  used when a build is being executed. The build packages and the runtime
  packages will be changed into separate lists so that the runtime environment
  is only installing the packages it needs.

* Ensure that 'optional' pip packages are installed into the venv during the
  build stage, rather than during the install stage.

.. _pypiserver: https://pypiserver.readthedocs.io

Alternatives
------------

* The build process can remain as-is, continuing to confuse deployers and
  difficult to maintain.

* The build process can be changed to only build and store wheels for packages
  which are pip installed onto the hosts, and only to build and store the
  venvs for distribution.

Playbook/Role impact
--------------------

Playbooks will be added to cater for the build process and the staging
process. The roles will be adjusted to properly separate out the build
tasks and the distro packages to install for the build (versus those
required when using pre-built wheels).

Upgrade impact
--------------

Care will be taken to ensure that upgrades happen as they do today.

Security impact
---------------

The security posture should be improved by the reduction of packages installed
onto hosts and containers when a full set of artifacts are built.

Performance impact
------------------

The performance of the deployment should be improved due to the reduction in
time taken to deploy with pre-built packages if a full set of artifacts are
built.

End user impact
---------------

There is no end-user impact for consumers of an OpenStack cloud, except
perhaps that upgrades will be quicker to execute, thus resulting in reduced
maintenance slot requirements.

Deployer impact
---------------

* As deployments and upgrades will be quicker to execute, deployers will be
  able to execute them in shorter maintenance slots.

* Deployers will need to understand how better to utilise the CI process to
  prepare the required artifacts to speed up deployments.

Developer impact
----------------

As the build process will be integrated into the roles, it will be easier to
understand how it works and what it does.

Dependencies
------------

This spec will be implemented in partnership with
https://blueprints.launchpad.net/openstack-ansible/+spec/deployment-stages

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  jesse-pretorius (odyssey4me)

Work items
----------

Each of the roles implemented in the default AIO will be worked through in
sequence to re-arrange and optimise based on this workflow. The work items
are not being detailed here but will be reflected in gerrit through the
blueprint's topic and will be visible in launchpad.

Testing
=======

As this process matures, it may be simpler to use the integrated build for
all role testing instead of having two seperate test implementations. This
reduces technical debt for the project.

Documentation impact
====================

This work will need to include documentation updates which describe the new
way that deployments can be implemented using full artifact builds and
how to implement offline installs.

References
==========

* https://12factor.net/

* http://www.clearlytech.com/2014/01/04/12-factor-apps-plain-english/


