Gate Split
##########
:date: 2015-09-07 12:00
:tags: gate, mitaka

The current integration gate check relies on an All-In-One (AIO) build which
is running low on resources and does not adequately test all code paths that
matter for the primary use-cases of the project.

This spec outlines a proposal to switch to using multiple gate checks which
are focused on testing multiple code paths that better reflect the primary
use-cases.

* https://blueprints.launchpad.net/openstack-ansible/+spec/gate-split


Problem description
===================

The current AIO gate check:

#. Is severely limited by the resources available in OpenStack-CI's 8 vCPU,
   8GB RAM per instance. While this is adequate for basic developer testing
   it is not a suitable reflection of the way deployments are done for
   production.

#. OpenStack-CI currently only provides for single- and two-node gate checks
   and have specifically asked that single-node checks be used as far as
   possible before implementing two-node checks.

#. Does not provide adequate code path coverage. It does not test the Ceph
   client configuration for Glance/Cinder, the NFS client configuration for
   Glance/Cinder, a standalone Swift deployment, or a deployment without
   Swift.

#. Tries to test as much as possible in one monolithic test, making the check
   difficult to understand, to maintain and to diagnose faults for.

#. Fails far too often. Reducing the container affinity as tested in
   https://review.openstack.org/221957 has identified that the resource
   constraints are most likely the primary reason for the regular tempest test
   failures in the HP Cloud provider of OpenStack-CI..


Proposed change
===============

Implement individual gate checks for OpenStack covering the following
use-cases using an AIO:

#. Compute with an NFS-backed Image and Block Storage service. This is a very
   commonly deployed design for environments with existing storage hardware
   investments. This AIO would be built with the following characteristics:
   - An NFS service on the host
   - Compute service on the host
   - HAproxy service on the host
   - Cinder built in a container, configured to use the NFS service
   - Glance built in a container, configured to use the NFS service
   - Single affinity for Keystone, Horizon, Galera, Repo, RabbitMQ containers
   - Ceilometer and Neutron deployed as in the AIO currently

#. Compute with a Ceph-backed Image and Block Storage service. This design is
   becoming more and more popular for deployments. This AIO would be built
   with the following characteristics:
   - Compute service on the host
   - HAProxy service on the host
   - An simple Ceph cluster running in three containers
   - Cinder built in a container, configured to use the Ceph service
   - Glance built in a container, configured to use the Ceph service
   - Single affinity for Keystone, Horizon, Galera, Repo, RabbitMQ containers
   - Ceilometer and Neutron deployed as in the AIO currently

#. Object Storage with Keystone. This is a typical 'Standalone Swift' design.
   For the sake of using the common infrastructure, we can add Glance to this
   for the purpose of verifying that Glance with a Swift back-end is still
   working correctly. This AIO would be built with the following
   characteristics:
   - HAProxy service on the host
   - Swift Account, Container and Object Storage on the host
   - Glance built in a container, configured to use Swift as a back-end
   - Single affinity for Keystone, Galera, Repo, RabbitMQ containers
   - Ceilometer deployed as in the AIO currently

#. Keystone Only. This is a specific gate test to verify the code paths for a
   cluster of three Keystone servers. This AIO would be built with the
   following characteristics:
   - HAProxy service on the host
   - 3 Keystone containers
   - Single affinity for Galera, Repo, RabbitMQ containers

#. Keystone with LDAP. This is a specific gate test to verify the code path
   for Keystone with an LDAP back-end. This AIO will be built with the
   following characteristics:
   - HAProxy service on the host
   - OpenLDAP on the host
   - 3 Keystone containers
   - Single affinity for Galera, Repo, RabbitMQ containers

#. Keystone with SSL. This is a specific gate test to verify the code path
   for Keystone with SSL enabled. This AIO will be built with the
   following characteristics:
   - HAProxy service on the host
   - 3 Keystone containers, with SSL enabled on Keystone's Apache
   - Single affinity for Galera, Repo, RabbitMQ containers

#. A high availability RabbitMQ cluster. This is to test both the deployment
   and the availability of the cluster when it's taken through a series of
   known failure scenarios. The details of the tests themselves would need to
   be clearly defined and implemented over time, so the gate check would start
   with what we have today - a simple test that the deployment works. The AIO
   would be built with the following characteristics:
   - Three RabbitMQ containers
   - A utility container for executing tests from

#. A high availability Galera cluster. This is to test both the deployment
   and the availability of the cluster when it's taken through a series of
   known failure scenarios. The details of the tests themselves would need to
   be clearly defined and implemented over time, so the gate check would start
   with what we have today - a simple test that the deployment works. The AIO
   would be built with the following characteristics:
   - HAproxy service on the host
   - Three Galera containers
   - A utility container for executing tests from

#. Repo Only. This is a specific gate test to verify the code paths for a
   cluster of three Repo servers and to take it through a series of known
   failure scenarios. The details of the tests themselves would need to be
   clearly defined and implemented over time, so the gate check would start
   with what we have today - a simple test that the deployment works. This AIO
   would be built with the following characteristics:
   - HAProxy service on the host
   - 3 Repo containers
   - A utility container for executing tests from

Each use-case gate check must have reference documentation covering the design,
the configuration implemented and the tests that are executed against it.

Also switch from our current lint check which combines Ansible syntax and lint
checks with python pep8 checks into the following checks which, where possible,
make use of the same OpenStack-CI jobs as are used by other projects:

#. bashate lint checks for bash scripts

#. pep8 lint checks for python scripts

#. Ansible syntax and lint checks for Ansible playbooks and roles


Alternatives
------------

Leave the current gate checks as they are.


Playbook/Role impact
--------------------

There will be no changes to the playbooks or roles as part of this work.


Upgrade impact
--------------

n/a

Security impact
---------------

n/a


Performance impact
------------------

n/a


End user impact
---------------

n/a


Deployer impact
---------------

n/a


Developer impact
----------------

#. More code paths will be tested.


Dependencies
------------

In order to implement variable load balancing configuration, this work depends
on: https://blueprints.launchpad.net/openstack-ansible/+spec/role-haproxy-v2


Implementation
==============

Assignee(s)
-----------

Primary assignee:
  https://launchpad.net/~jesse-pretorius ``odyssey4me``

Other contributors:
  https://launchpad.net/~hughsaunders ``hughsaunders``


Work items
----------

For each use-case:

#. Develop and document the design.

#. Implement a non-voting experimental gate check.

#. Push the code and documentation up for review and use 'check experimental'
   to validate its functionality.

#. Switch the gate check to the normal check queue, leaving it as non-voting,
   in order to do final functional validation.

#. Switch the gate check to voting and add it to the merge queue.


Testing
=======

Please see 'Work items'.

Documentation impact
====================

As indicated in the proposed change, each gate check should be properly
documented for easier reference and understanding.


References
==========

None.
