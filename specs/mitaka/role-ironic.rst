Role Ironic
###########
:date: 2015-10-12 16:30
:tags: ansible, ironic

The purpose of this spec is to add support for the OpenStack Ironic program
to OpenStack Ansible, allowing the provisioning of compute nodes to bare metal
machines.

https://blueprints.launchpad.net/openstack-ansible/+spec/role-ironic


Problem description
===================

Openstack Ansible currently does not support the provisioning of bare metal
compute hosts, but this is functionality that operators and users are likely
to want.


Proposed change
===============

The Ironic program encompasses a number of projects, but this spec and this
proposed series of changes covers the initial implementation of support for
Ironic. This will involve adding support for the Ironic server[1] and Ironic
client[2].

Future specs may be raised to cover the addition of ironic-inspector, or to
support alternate deployment mechanisms, or to support different deployment
drivers. The specific detail for these will be added in future specs.

This work will build upon the experiences learnt in developing bifrost[3]
(which is a set of ansible playbooks for deploying Ironic standalone, without
other OpenStack components).

The changes that are proposed as part of this spec are:

* Creation of an openstack-ansible-ironic repository and ansible role to
  support the initial implementation of Ironic.  This will allow
  openstack-ansible to deploy compute nodes to bare metal hosts, via the nova
  API. Initially, support will be limited to bare metal hosts that support
  IPMI for power control, and PXE for boot.

* Tests to verify the new ansible role


Alternatives
------------

None, really.  Supporting bare metal hosts in OpenStack is done via using
Ironic.


Playbook/Role impact
--------------------

Test playbooks will be placed in the openstack-ansible-ironic repository
for functional testing purposes, with no initially proposed changes to
openstack-ansible playbooks.

In the future, once the ironic role is deemed useful and acceptible, a future
spec will address the integration of the ironic role with the main
openstack-ansible repository.


Upgrade impact
--------------

None


Security impact
---------------

None.


Performance impact
------------------

None.


End user impact
---------------

Deployers will be able to deploy compute nodes to bare metal hosts.


Deployer impact
---------------

Ironic specific configuration options will be added to the new repository.
When support for the new Ironic role is added to the parent repository new
config options will be made available, however it is expected that Ironic
support will initially be disabled, requiring that deployers explicitly
enable Ironic support, and to enrol hosts for openstack-ansible to use.


Developer impact
----------------

As this change is self-contained initially, no impact on other developers
is expected.


Dependencies
------------

None


Implementation
==============

Assignee(s)
-----------

Primary assignee:
  Michael Davies - mrda on Launchpad and on IRC

Other contributors:
  None


Work items
----------

#. Ask for the new repository, openstack-ansible-ironic, to be created
#. Create the role for ironic support

   * Add support for running ironic-api
   * Add support for running ironic-conductor
   * Add support for including python-ironicclient, which is the operator
     tool for supporting Ironic.
   * Add configuration to make configuring bare metal deployment easy
#. Add support for enrolling bare metal nodes
#. Add support for configuring Nova to use Ironic.  Initially this will be in
   the form of documentation until the parent openstack-ansible repository is
   updated to use openstack-ansible-ironic


Testing
=======

As this is testing deploying to hardware, this is challenging :)

Develop a test playbook to deploy to hardware that can exercise the new
role.  Develop tests that verify the role's behaviour independent of
actually requiring hardware to test the role's functionality.


Documentation impact
====================

Adding support to the user guide on how to enable Ironic support will be
required.


References
==========

* [1] The Ironic server: http://git.openstack.org/cgit/openstack/ironic/
* [2] The Ironic client:
  http://git.openstack.org/cgit/openstack/python-ironicclient/
* [3] The Bifrost project, standalone Ironic installation:
  http://git.openstack.org/cgit/openstack/bifrost
