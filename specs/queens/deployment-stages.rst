Implement deployment stages for optimised execution
###################################################
:date: 2017-09-14 12:00
:tags: optimise, lifecycle

In order to improve ease of use, optimise execution and provide the ability to
make use of pre-built artifacts in deployments this spec proposes the
implementation of deployment stages.

  * https://blueprints.launchpad.net/openstack-ansible/+spec/deployment-stages

Problem description
===================

* In production environments with many target hosts there are sometimes
  transient failures that happen. When they happen the deployer is forced to
  re-execute playbooks which may go through many tasks which are already
  complete and do not need to be executed again. While a knowledgable deployer
  will make use of tag skipping and host scoping to reduce the execution time,
  this is not a skill the novice deployer has. In order to improve ease-of-use
  it should be possible for the playbooks to simply skip over the stages which
  have already completed on each host.

* In production environments it may be desired to make use of a fully
  artifacted deployment in order to ensures that multiple regions are deployed
  using exactly the same software. Currently there is no tooling included to
  facilitate the complete stack of artifacts (apt, git, python, container)
  that need to be built.

* Deployments currently do a lot of outgoing internet interaction in order
  to fetch packages, keys and other artifacts. The outgoing access is often
  a problem for deployers with a high security environment as the hosts are
  not able to access the internet directly. This access is also slower than
  it would be if these artifacts were locally staged before deployment.

* Deployments currently mix the build of artifacts with their installation
  and activation. This results in very long deployment times which often
  exceed maintenance periods available for operations. If the artifact build
  process could be executed and the artifacts could be staged without
  operationally impacting a production environment, then these could be
  executed prior to a maintenance slot and only the final step of implementing
  changes to use the new artifacts could be done in the maintenance slot.

* Deployments currently do a lot of staging actions in serial due to the
  combined install/config tasks in each role. This takes a very long time
  and is not necessary. If the build and stage tasks are properly split from
  the configuration changes then the build/stage tasks could be executed in
  parallel and only the configuration changes executed in serial,
  significantly speeding up large deployments.

Proposed change
===============

The stages proposed are as follows:

#. Build: This stage prepares artifacts which are general purpose. This stage
   could be executed by a CI process in order to prepare the appropriate
   artifacts and stored on a server to be used across multiple regions.
   Alternatively it could be executed in-line for a single build (using
   'developer_mode'. Artifact examples include distribution software packages,
   container rootfs tarballs, python venvs, etc. If not executed in-line, the
   build process should be executed on any designated host and produce
   artifacts which can be copied to a web server. There must be a well defined
   manifest detailing the artifacts produced which can easily be used for a
   staging process to understand which items to fetch.

#. Stage: This stages all artifacts from the Build stage using the manifest
   produced. The stage is optional and will only be executed if the Build
   stage was executed to build all artifacts. The stage will most likely only
   be a playbook rather than something in the role, making it easy to allow
   deployers to implement alternative staging mechanisms if they choose to.
   This stage will be executed in parallel across all hosts/containers to
   ensure that it executes quickly.

#. Install: This stage executes the code path which uses the staged or built
   artifacts and the prepared OSA configuration to create containers and
   install all services. This process should not restart containers or services
   or enact any changes to an existing environment which will disrupt it. This
   stage will be executed in parallel across all hosts/containers to ensure
   that it executes quickly.

#. Configure: This stage executes the implementation of configuration changes
   to configuration files and starts/restarts the applicable services or
   containers. This stage will be executed serially to ensure that service
   disruption is minimised.

The tasks for each stage will be explicitly broken into task files, for example:

* <service>_build.yml
* <service>_install.yml
* <service>_install_apt.yml
* <service>_install_nginx.yml
* <service>_configure.yml
* <service>_configure_nginx.yml
* <service>_configure_ssl.yml
* <service>_configure_keys.yml

The general idea with breaking out the task files is to implement conditional
and/or dynamic inclusions where appropriate to ensure that the tasks are not
even evaluated unless a broad condition is met. This is different to having a
bunch of tasks in a single file which all have conditions because Ansible will
not have to evaluate each task in turn, but instead evaluate whether a block of
tasks should be evaluated. This reduces execution time.

Some examples:

#. If pre-built artifacts are available when the role executes, skip the
   build stage tasks.
#. If there is no repo server in the environment, do not try to download
   any python venvs or other artifacts.
#. If ``ansible_pkg_mgr == 'apt'``, do not evaluate any tasks related to
   yum.

As part of this solution, the build and install stages should drop local facts
on to target hosts when the stage completes. The local fact will prevent that
stage being executed again through a conditional include. This provides a
checkpoint restart mechanism so that if a deployer executes 'setup-everything'
the execution will be much faster because it will skip whole stages and
continue from where it left off. This also means that if pre-built artifacts
are used, these stages will be skipped and the deployment in an environment
will be much, much quicker.

The facts dropped would be tag-specific - for example the fact dropped would
indicate that the 'cinder' service has the '14.2.0' release installed on the
host, meaning that the build and staging tasks do not need to be run if the
proposed tag and the tag deployed are the same. This behaviour will be
overridable via another variable which enables a forced rebuild or forced
reinstall.

Alternatives
------------

#. Put up with long deployment times.

#. Document in better detail how to reduce deployment times using package
   mirrors, proxies and such.

Playbook/Role impact
--------------------

New playbooks will be implemented which allow the deployer to executed the
more targeted build process and to prepare the artifacts. The existing
playbooks will continue to work, but will be adjusted to make use of the
appropriate facts to skip the previously executed build process if that has
already been executed.

The roles will be where the greatest impact will be as many of the tasks will
be re-organised to facilitate the staged process.

Upgrade impact
--------------

Being able to make use of pre-built artifacts for an environment will mean
that an upgrade process should be able to more easily roll back to a
previous state if need be.

Security impact
---------------

As this process will improve the ability to ensure a consistently built
environment, this will likely improve the security posture of a deployment.

Performance impact
------------------

Hopefully the deployment and upgrade performance will be far better than
it is now. The running deployment performance should be no different.

End user impact
---------------

There will be no difference to end-users of the deployed OpenStack
environment.

Deployer impact
---------------

Deployers will continue to have the same entry points, but will gain the
ability to pre-build artifacts for their environment in order to ensure
that deployments and upgrades execute more quickly and reliably.

Developer impact
----------------

These changes should improve the developer experience by reducing the time
taken to implement an AIO.

Dependencies
------------

None

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

It may be possible for us to make use of pre-built artifacts for gate testing
in order to reduce the time take for integrated tests. The option of
publishing the last successful build's artifacts for each branch on OpenStack
Infrastructure will be explored. These artifacts will be for development tests
only and not useful for production environments.

Documentation impact
====================

The staged deployment process will need to be documented and the details of
how to opt-in to make use of an artifacted build will need to be included.

References
==========

None

