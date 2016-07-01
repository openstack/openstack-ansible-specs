Multiple CPU Architecture Support
#################################
:date: 2016-06-14 12:00
:tags: openstack, ansible, power

The purpose of this spec is to enable OpenStack-Ansible to deploy OpenStack
in clouds with multiple CPU architectures across the nodes to be deployed.

https://blueprints.launchpad.net/openstack-ansible/+spec/multi-arch-support

The purpose of this spec is to add support for multiple CPU architectures to
OpenStack-Ansible. With the introduction of OpenStack services running on other
architectures such as PPC64LE, OpenStack-Ansible needs to be extended to support
environments where a mixed environment of CPU architectures exists.


Problem description
===================

OpenStack-Ansible was initially built to support deployments to a single
architecture, primarily x86. With the extension of OpenStack and
OpenStack-Ansible support to other platforms such as POWER, support for
deployments running a combination of different CPU architectures is needed.

For each deployment, OpenStack-Ansible creates and builds a 'repo' containing
necessary artifacts for the OpenStack deployment. This repo holds components
such as pip wheels, virtualenvs, and source trees for the different services
to be deployed. For each deployment a single 'master' repo is designated where
artifacts are built, then synchronized out to the rest of the slaves.

This creates problems in a multi-architecture deployment for nodes where the
repo master's CPU architecture is different than the architecture of other
nodes. For example, deployment of a KVM on POWER compute node will fail when
the artifacts were built on an x86 repo master used for the control plane.


Proposed change
===============

Update OpenStack-Ansible and necessary roles to support building and deploying
with multiple CPU architectures. This includes changes to:

* Look at the Ansible facts for all hosts and determine the set of CPU
  architectures to build artifacts for.

* Add support for assigning a build master for each CPU architecture.

* Support building copies of CPU architecture specific artifacts on each build
  master, which will be synchronized to all slaves regardless of architecture.

* Support tagging all architecture-specific build artifacts with the
  corresponding CPU architecture.

* Ensure pre-built binaries used during installation (apt packages, etc) exist
  or are made available for the supported CPU architectures (x86, ppc64el)
  where possible, or documentation added to note limitations where not
  available.

Alternatives
------------

* Cross-compiling artifacts - This would remove the need for building, tagging
  and synchronizing between repos built on different architectures, but so far
  a reliable way to do this for wheels/venvs has not been found. This also
  introduces increased risk of architecture-specific cross-compiling issues
  as support for additional architectures is added.

* Support only single-architecture OpenStack-Ansible deployments. This is
  harmful to deployers by limiting the potential integration of servers
  from other architectures into new and existing OpenStack-Ansible deployments

Playbook/Role impact
--------------------

There will be impact to the repo playbooks to handle building and
synchronizing across multiple architectures. There will also be minor
impact to each role to support building and tagging artifacts with the
required architecture information.

Upgrade impact
--------------

N/A

Security impact
---------------

N/A

Performance impact
------------------

Both repo build and synchronization operations will take longer to complete
proportionate to the number of CPU architectures being deployed.

* Repo builds will take longer as artifacts for each repo must be built
  serially to avoid collisions during synchronization of non-architecture
  specific packages.

* Synchronizations will take longer and require more bandwidth to transfer
  when there are multiple architectures due to the increased number of
  artifacts being built.

End user impact
---------------

No end user impact is expected.

Deployer impact
---------------

Deployers will now be able to create deployments with servers of
multiple CPU architectures included.

No other deployer impact is expected as the detection of multiple
architectures, deploy of the required number of repo masters for each
architecture, and build of artifacts for each architecture should happen
dynamically.


Developer impact
----------------

Developer impact will be minor. Developers will now be able to develop for
and test deployments across multiple CPU architectures. It also enables
development of roles specific to other CPU architectures in the future, such
as ARM.


Dependencies
------------

N/A

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  ashana@us.ibm.com/ashana

Other contributors:
  adreznec@us.ibm.com/adreznec
  thorst@us.ibm.com/thorst

Work items
----------

* Add a repo discovery task to determine the CPU architectures used across
  all hosts, and assign and store facts about the master for each
  architecture.

* Modify the repo-server playbook to support deploying a build master for each
  CPU architecture.

* Modify the repo-build playbook to support building repos for each CPU
  architecture in serial.

* Update the build tasks to support tagging each artifact with a corresponding
  CPU architecture.

* Update the repo synchronization to support synchronizing artifacts from each
  build master out to all other slaves, regardless of CPU architecture.

* Updates across tasks to use the new tagged artifact names.

* Ensure binary packages are available on all package mirrors/locations for
  ppc64el in addition to i386/amd64, or that documentation is added to
  note where packages aren't available.

Testing
=======

A new test job will be added that deploys a two-node configuration, with
one node belonging to the upstream/default architecture (x86) and the other
of an additional CPU architecture to be supported (ppc64el, arm64, etc).

Documentation impact
====================

Documentation covering how to configure multi-arch support will be added to
the user guide.

References
==========

* http://specs.openstack.org/openstack/openstack-ansible-specs/specs/newton/powervm-virt-driver.html
