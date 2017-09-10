Generalize Infrastructure Roles
###############################
:date: 2017-09-10 14:00
:tags: ansible, roles, mariadb, rabbitmq

Provide a synopsis as to why you are creating this spec/blueprint.

Include the URL of your launchpad blueprint:
  * https://blueprints.launchpad.net/openstack-ansible/+spec/ansible-roles-reuse

Currently openstack-ansible is maintaining infrastructure roles that are used
to deploy general infrastructure services such as MariaDB and RabbitMQ, which
are applicable in non-OpenStack ansible environments also. With little to no
refactoring these roles can be used to deploy the services in other Ansible
managed environments also.

By maintaining robust, generalized service roles, they are more likely to be
consumed, improved, and maintained by other operators in the greater Ansible
community. This will benefit us by training us to keep a modular mindset when
building the roles, which leads to better maintainability and wider testing
for OSA consumers also.

In some cases we may wish to deprecate our openstack-ansible roles and consume
more generalized upstream alternatives.


Problem description
===================

In some of the roles (such as haproxy), we implement a very OSA specific
deployment with very little reusability or configurability for a typical
HAProxy deployer.

Other roles, such as Galera server, are fairly generalized and robust, but
carry the openstack-ansible-service_name naming scheme, making it less likely
for anyone NOT using openstack-ansible to use the role in their deployments.

pip_install is an example of a role that will require some minor refactoring to
generalize it. The role performs some very out of scope tasks, such as repo
management, which have nothing to do with installing pip. These features should
be moved to appropriately modularized roles (a general repo management role?),
so that pip_install is only doing the work it is meant to do.


Proposed change
===============

Examine the following roles to identify and refactor out of scope tasks and
orchestrate openstack-ansible specific configurations at the integrated repo
level. If the role is built properly it should offer the necessary service
configuration to be injected from the inventory and playbooks.

Roles to examine initially:
  * openstack-ansible-pip_install
  * openstack-ansible-lxc_hosts
  * openstack-ansible-lxc_container_create
  * openstack-ansible-haproxy_server
  * openstack-ansible-memcached_server
  * openstack-ansible-galera_server
  * openstack-ansible-rabbitmq_server
  * openstack-ansible-ceph_client

Once the work outlined above has progressed sufficiently, we should consider
renaming some of the roles to a more appropriate naming, ie.
openstack-ansible-galera_server becomes ansible-mariadb-cluster, etc.


Alternatives
------------

N/A

Playbook/Role impact
--------------------

The playbooks and especially inventory should eventually contain all of our
openstack-ansible specific configurations. The infrastructure roles themselves
should be generalized without an assumption or skew toward being consumed only
by openstack-ansible.

In some cases this is already implemented, but in other cases the role will
undergo significant changes or wholesale replacement to accomplish this.


Upgrade impact
--------------

Consumers of the roles will need to adjust to any major refactorings that take
place, including possible renaming of the git sources and role names.


Security impact
---------------

N/A


Performance impact
------------------

N/A


End user impact
---------------

N/A


Deployer impact
---------------

Deployers who work frequently with openstack-ansible will benefit from the
ability to use the same roles to deploy applicable services for other projects
they work on besides OSA.


Developer impact
----------------

It is possible that this could draw more developers to assist in maintaining
some of the roles. Cosmetic changes such as renaming may also help veteran
OSA developers take a more abstract approach when crafting changes to these
roles, which should make them more maintainable in the long run.


Dependencies
------------

N/A


Implementation
==============

Assignee(s)
-----------

Primary assignee:
  Logan Vig (LP: loganv; IRC: logan-)


Work items
----------

* Examine the infrastructure roles for out of scope tasks or reusability
  concerns. Address the issues by refactoring or replacing the role.
* Improve the role documentation if necessary with example playbooks
  demonstrating ad-hoc usage of the role.
* Rename the role and repo to a globally namespaced ansible role such as
  ansible-service-name.


Testing
=======

N/A


Documentation impact
====================

Improving and expanding the role documentation will be beneficial for
reusability also.


References
==========

* openstack-ansible 5/18 community meeting: http://eavesdrop.openstack.org/meetings/openstack_ansible_meeting/2017/openstack_ansible_meeting.2017-05-18-16.01.log.html#l-136
