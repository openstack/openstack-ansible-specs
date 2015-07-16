Compartmentalize RabbitMQ
#########################
:date: 2015-07-14
:tags: rabbitmq

The purpose of this spec is to adjust our current RabbitMQ setup to better use
the available system resources by creating a vhost and user per-consumer
service within RabbitMQ.

Include the URL of your launchpad blueprint:
  * https://blueprints.launchpad.net/openstack-ansible/+spec/compartmentalize-rabbitmq


Problem description
===================

Presently all services use the single root virtual host within RabbitMQ and while this
is "OK" for small to mid sized deployments however it would be better to divide
services into logical resource groups within RabbitMQ which will bring with it
additional security.


Proposed change
===============

All services that utilize RabbitMQ should have their own virtual host, user, and
password.

Overview:
  * Each role would use the upstream Ansible RabbitMQ user module to create a new
    user. The username will be customizable with a default being the same as the
    name of the service.
  * Each role will use the upstream Ansible RabbitMQ vhost module to create a new
    virtual host per service. The vhost will be customizable with a default being
    the same as the name of the service.
  * A Password entry will be created within the ``user_secrets.yml`` file for
    each RabbitMQ service user.
  * The oslo config section of each service will be updated to use the new vhost
    name, username, and password.


Alternatives
------------

Leave RabbitMQ the way it is.


Playbook impact
---------------

The playbooks will have no impact. The changes being proposed are being done
within roles. Ideally this would be a simple default addition, two new tasks,
and a simple change within the oslo_messaging section in the service
configuration files.


Upgrade impact
--------------

There will be an upgrade impact as the user will need to add the new secret
entries to the ``user_secrets.yml`` file. If this was to be accepted as a
backport to kilo this would have to be targeted to a major version.


Security impact
---------------

Serpentining the services into different vhosts with different users and passwords
should improve security. And brings our project more inline with what is described
in the OpenStack Messaging Security documentation.

* http://docs.openstack.org/security-guide/content/messaging-security.html


Performance impact
------------------

The separation of service into logical vhosts has been not been reported to have
any noticeable performance impact.

* http://stackoverflow.com/questions/12518685/
  performance-penalty-of-multiple-vhosts-in-rabbitmq
* http://lists.rabbitmq.com/pipermail/rabbitmq-discuss/2012-September/
  022618.html


End user impact
---------------

n/a


Deployer impact
---------------

The deployer will need to ensure they have passwords entries set within the
``user_secrets.yml`` file. This should not impact greenfield deployments however
it will need to be something covered in an upgrade.


Developer impact
----------------

n/a


Dependencies
------------

n/a

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  https://launchpad.net/~kevin-carter ``cloudnull``


Work items
----------

* Add new RabbitMQ users for all services.
* Add new RabbitMQ vhosts for all services.
* Update all service configuration files to use the new vhost, user,
  and password.


Testing
=======

The testing of this change is a convergence test. The gate job will utilize the
the changes on every commit.


Documentation impact
====================

Docs will need to be updated in terms of upgrades to add the new variables.


References
==========

n/a
