Additional Role for Tacker Service Deployment
##############################################
:date: 2016-10-19 12:30

:tags: tacker, openstack-ansible

The purpose of this spec is to add support for the OpenStack Tacker service
to OpenStack-Ansible. This would allow the deployment of Tacker along with
the core OpenStack components using OpenStack-Ansible.

Blueprint - Tacker deployment on OpenStack-Ansible:

https://blueprints.launchpad.net/openstack-ansible/+spec/role-tacker


Problem description
===================

Presently, while deploying OpenStack using OpenStack-Ansible only the core
OpenStack components get deployed. The deployment of other components
(eg: Tacker) on playbooks is not supported yet and to use other
component's services, they need to be deployed manually.


Proposed change
===============

This change involves adding support for the Tacker server, Tacker client,
and Tacker Horizon dashboard interface.

The proposed changes include:

* Creation of an openstack-ansible-tacker repository and Ansible role
  to support the deployment of Tacker.
* Tests to verify the new Ansible role.
* Deployment of Tacker client
* Deployment of Tacker Horizon


Alternatives
------------

None


Playbook/Role impact
--------------------

Test playbooks will be placed in the openstack-ansible-tacker repository
for functional testing purposes, with no initially proposed changes to
OpenStack-Ansible playbooks.

In the future, once the Tacker role has reached a muture state, a future
spec will address the integration of the Tacker role with the main
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

Deployers will be able to deploy Tacker service through OpenStack-Ansible
framework for VNF management and orchestration purposes.


Deployer impact
---------------

When support for the new Tacker role is added to the parent repository, new
Tacker specific configuration options will be made available. This will
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
  Jeff Rametta ( IRC: jcrst)

Other contributors:
  None


Work items
----------

#. Ask for the new repository, openstack-ansible-tacker, to be created
#. Create the role for Tacker support

   * Add support for running tacker-sever
   * Add support for python tacker client
   * Add support for Tacker Horizon dashboard
   * Add documentation and install guide for the role


Testing
=======

The usual gate checks can be used for these changes. Also, each individual
commit can be functionally tested individually.


Documentation impact
====================

Adding support to the user guide on how to enable Tacker support will be
required.

References
==========

* Tacker server: https://git.openstack.org/cgit/openstack/tacker/
* Tacker client: https://git.openstack.org/cgit/openstack/python-tackerclient
* Tacker Horizon: https://git.openstack.org/cgit/openstack/tacker-horizon


