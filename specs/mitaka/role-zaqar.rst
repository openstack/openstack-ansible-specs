Additional Role for Zaqar Deployment
########################################
:date: 2016-01-20 11:20

:tags: zaqar, openstack-ansible

The purpose of this spec is to add support for the OpenStack Zaqar program
to OpenStack-Ansible. This would allow the deployment of Zaqar along with
the core OpenStack components using OpenStack-Ansible.

Blueprint - Zaqar deployment on OpenStack-Ansible:

https://blueprints.launchpad.net/openstack-ansible/+spec/role-zaqar


Problem description
===================

Presently, while deploying OpenStack using OpenStack-Ansible only the core
OpenStack components get deployed. The deployment of other components
(eg: Zaqar) on playbooks is not supported yet and to use other
component's services, they need to be deployed manually.


Proposed change
===============

The Zaqar program encompasses a number of projects, but this spec and this
proposed series of changes covers the initial implementation of support for
Zaqar. This will involve adding support for the Zaqar server[1] and
Zaqar client[2].

The proposed changes include:

* Creation of an openstack-ansible-zaqar repository and Ansible role
  to support the deployment of Zaqar.
* Tests to verify the new Ansible role.


Alternatives
------------

None


Playbook/Role impact
--------------------

Test playbooks will be placed in the openstack-ansible-zaqar repository
for functional testing purposes, with no initially proposed changes to
OpenStack-Ansible playbooks.

In the future, once the Zaqar role is found to be useful and acceptable, a
future spec will address the integration of the Zaqar role with the main
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

Deployers will be able to deploy Zaqar and use messaging service through
OpenStack-Ansible.


Deployer impact
---------------

When support for the new Zaqar role is added to the parent repository, new
Zaqar specific configuration options will be made available. This will
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
  Fei Long Wang ( IRC: flwang)

Other contributors:
  None


Work items
----------

#. Ask for the new repository, openstack-ansible-zaqar, to be created
#. Create the role for Zaqar support

   * Add support for running zaqar-sever
   * Add support for including python-zaqarclient, which is the operator
     tool for supporting Zaqar.


Testing
=======

The usual gate checks can be used for these changes. Also, each individual
commit can be functionally tested individually.


Documentation impact
====================

Adding support to the user guide on how to enable Zaqar support will be
required.

References
==========

* [1] The Zaqar server: http://git.openstack.org/cgit/openstack/zaqar/
* [2] The Zaqar client:
  http://git.openstack.org/cgit/openstack/python-zaqarclient/
