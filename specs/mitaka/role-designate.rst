Additional Role for Designate Deployment
########################################
:date: 2015-12-08 12:00

:tags: designate, openstack-ansible

The purpose of this spec is to add support for the OpenStack Designate program
to OpenStack-Ansible. This would allow the deployment of Designate along with
the core OpenStack components using OpenStack-Ansible.

Blueprint - Designate deployment on OpenStack-Ansible:

https://blueprints.launchpad.net/openstack-ansible/+spec/role-designate


Problem description
===================

Presently, while deploying OpenStack using OpenStack-Ansible only the core
OpenStack components get deployed. The deployment of other components
(eg: Designate, Trove) on playbooks is not supported yet and to use other
component's services, they need to be deployed manually.


Proposed change
===============

The Designate program encompasses a number of projects, but this spec and this
proposed series of changes covers the initial implementation of support for
Designate. This will involve adding support for the Designate server[1] and
Designate client[2].

The proposed changes include:

* Creation of an openstack-ansible-designate repository and Ansible role
  to support the deployment of Designate.
* Tests to verify the new Ansible role.


Alternatives
------------

None


Playbook/Role impact
--------------------

Test playbooks will be placed in the openstack-ansible-designate repository
for functional testing purposes, with no initially proposed changes to
OpenStack-Ansible playbooks.

In the future, once the Designate role is found to be useful and acceptable, a
future spec will address the integration of the Designate role with the main
OpenStack-Ansible repository.


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

Deployers will be able to deploy Designate and use DNSaaS through
OpenStack-Ansible.


Deployer impact
---------------

When support for the new Designate role is added to the parent repository, new
Designate specific configuration options will be made available. This will
provide an optional role for use in the OpenStack-Ansible toolbox for the
deployers.


Developer impact
----------------

As this change is self-contained initially, no impact on other developers is
expected.


Dependencies
------------

None


Implementation
==============

Assignee(s)
-----------

Primary assignee:
  Swati Sharma ( IRC: Swati)

Other contributors:
  None


Work items
----------

#. Ask for the new repository, openstack-ansible-designate, to be created
#. Create the role for Designate support

   * Add support for running designate-api, designate-central,
     designate-pool_manager, designate-sink, designate-mdns
   * Add support for including python-designateclient, which is the operator
     tool for supporting Designate.


Testing
=======

The usual gate checks can be used for these changes. Also, each individual
commit can be functionally tested individually.


Documentation impact
====================

Adding support to the user guide on how to enable Designate support will be
required.

References
==========

* [1] The Designate server: http://git.openstack.org/cgit/openstack/designate/
* [2] The Designate client:
  http://git.openstack.org/cgit/openstack/python-designateclient/
