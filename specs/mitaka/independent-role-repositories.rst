Independent Role Repositories
#############################
:date: 2015-08-17 14:00
:tags: roles

In order to improve the ability to independently consume the roles produced by
openstack-ansible in different reference use-case deployments and allow
independent development of each role by different projects, this specification
proposes that:

#. New roles be registered in separate repositories named
   ``openstack/openstack-ansible-<role>``.

#. Existing roles can, through an independent blueprint/spec process, be split
   into their own repositories.

* https://blueprints.launchpad.net/openstack-ansible/+spec/independent-role-repositories

This provides the following benefits:

* Other projects (eg: DevStack, Kolla, Compass, RPC, etc) will be able to
  consume the roles using their own playbooks. This increases the opportunity
  for other projects to collaborate with openstack-ansible.

* The roles will be more easily consumed in different reference architectures.
  Currently the playbooks and roles are specifically geared for deployment in
  LXC containers. Making the roles independent entities will allow them to be
  consumed for completely different architectures. eg: a deployment without
  containers, a deployment with VM's instead of containers, a deployment with
  a different container technology.

* Each role can be developed at its own pace and versioned independently. This
  will make openstack-ansible more like the OpenStack "Big Tent".

* Each role can be independently gate checked with checks that are specific
  and relevant to the role. This will provide quicker developer feedback and
  allow a quicker turnaround for development.

* The roles can be registered in Ansible Galaxy. This will provide greater
  awareness of the roles and may attract more contributors. This is especially
  useful for the infrastructure roles which may have a broader application
  than just for use in an OpenStack deployment. (eg: haproxy, MariaDB, etc)

* Role separation will simplify the mission of openstack-ansible's playbooks
  and scripts to be for the purpose of providing examples of how to consume
  the roles and to implement gate testing for integrated deployment
  verification of the use-cases that matter to the community. This prevents
  the situation where the playbooks have to cater for every possible
  combination use-case that may be thrown at them.

Problem description
===================

A detailed description of the problem:

* Currently the openstack-ansible roles are tightly coupled with the
  playbooks that consume them. While the roles can technically be
  consumed using different playbooks, this is not immediately
  obvious to downstream consumers.

* When consumers to try to consume the roles with different playbooks
  for a different architecture, they are forced to implement many
  workarounds for the tight coupling that we have. Even when it is
  possible to do, it is hard for a deployer to see how to do it due
  to the deluge of variables that need to be set and code that needs
  to be read.

* There has been some interest in creating roles for other services
  (eg: rally, congress, etc). Implementing the strategy of a repo
  per role allows these fledgling roles to get into the open and
  get collaborated on far more quickly. Once they're at the point
  where they're ready to be integrated into an integrated use-case
  with integration gate tests, then they can tag a version and
  implement the openstack-ansible playbooks and scripts to test
  the appropriate use-case.

* The infrastructure roles implemented as part of the project are not
  getting much attention as they are more like a second-class citizen
  within the role structure. This results in the configurations
  deployed often not lining up to best-practices.

* Any small changes to roles require the execution of a full
  integration gate test which is slow and prone to error. It is
  difficult to isolate the error in the current monolithic stack.


Proposed change
===============

The following documentation should be developed:

#. The primary use-cases/implementations tested by the project.

#. How to apply to add a new use-cases for integration testing.

#. How to register a new role within openstack-ansible's umbrella.

#. First steps for building a new role.

#. Update the README for each role to describe how to use it independently,
   whether using a static inventory or the dynamic inventory. It should also
   cover what options are available for its use, whether it relies on any
   other roles, how upgrades are handled, any known issues, etc.

The following process should be followed for registering new roles:

#. A blueprint must be registered and a spec for the implementation provided
   for review, with specific attention paid to any changes that would need to
   be made in the current openstack-ansible playbooks and roles to integrate
   the new role.

#. Once the spec is approved, the review to register the new repository should
   be registered upstream by the openstack-ansible PTL or a nominated
   openstack-ansible-core team member.

#. Once the new repository has been created, work can commence on the new
   role.

#. Before the first tag is set for the role, comprehensive testing for the
   role must already be in place.

The following process should be followed for breaking out existing roles:

#. For each role targeted for breaking out, a separate blueprint must be
   registered and a spec provided for the high level changes that would be
   required in the current openstack-ansible playbooks and roles to
   accommodate this change.

#. Once the spec is approved, the review to register the new repository should
   be registered upstream by the openstack-ansible PTL or a nominated
   openstack-ansible-core team member.

#. Once the new repository has been created, work can commence on extracting
   the role as planned in the spec for the role's migration.

#. Once independent gate checks on the role repository confirm that it is
   in working order and the work is done to prepare the role for usage in
   the integrated use-cases, tag the initial version of the role and
   implement the openstack-ansible playbook, script and role-requirements
   changes to consume the new role. The changes in openstack-ansible will
   need to pass the integrated gate checks before they can merge.


Alternatives
------------

Leave everything as it is and continue to merge any new roles proposed into
the same monolithic repository.


Playbook/Role impact
--------------------

The impact to playbooks should be fairly incremental, but will need to be
determined on a role by role basis. This must be described in the spec on a
per role basis.

It is clear that the libraries, filters and plugins will need to be broken out
into its own repository and each role that consumes them will need to use a
submodule reference in the role. This ensures that all the roles use a common
set of libraries, filters and plugins.


Upgrade impact
--------------

There are two aspects of upgrade impact to be considered:

#. The ability for the role itself to handle upgrades from previous
   interations of itself.

#. The ability for the integrated build use-cases that consume the
   role to be upgradable.

Each role should be developed with upgradability in mind and conform
to the upgrade framework which is being developed for the Kilo -> Liberty
upgrade process.


Security impact
---------------

n/a


Performance impact
------------------

n/a


End user impact
---------------

End users will not know of any differences.


Deployer impact
---------------

The structure and placement of roles on the deployment host will be
different. The changes will need to be documented for support
purposes. However, the configuration and execution of the existing
playbooks for downstream consumers should be targeted to be exactly
the same to minimise disruption.

Any role-specific impacts will need to be defined on a per-role
basis.


Developer impact
----------------

The biggest negative developer impact will be the difficulty working
between the two different structures - master being split, with
liberty and kilo being consolidated. It may be worth considering
ways to make them all work the same way once the conversions are
done for master.

The positive impact has been outlined in the introduction.


Dependencies
------------

This should only be implemented after liberty has been released.


Implementation
==============

Assignee(s)
-----------

Primary assignee:
  https://launchpad.net/~jesse-pretorius ``odyssey4me``
  https://launchpad.net/~kevin-carter ``cloudnull``

Other contributors:
  TBD

Work items
----------

See proposed change section.

Testing
=======

The testing impact will need to be described on a per-role basis.

As a standard, the following tests are expected to be implemented as
a standard for each role:

* bashate tests for all shell scripts
* pep8 tests for all python files
* ansible syntax checks for all ansible task files
* ansible-lint tests for all ansible task files
* functional tests for the service

Integration tests are expected to be implemented in the openstack-ansible
repository and executed whenever the role versions are incremented. This
ensures that a role tag increment is only accepted for an integrated
release once it passes a full set of integration tests.


Documentation impact
====================

While the placement of the role files on the deployment host will be
different, the configuration and execution of the deployment should
remain the same, resulting in minimal documentation impact.

See the proposed change section for developer reference documentation
to be developed.


References
==========

n/a
