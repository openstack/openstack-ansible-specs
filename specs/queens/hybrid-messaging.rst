Provide option of hybrid messaging backends
###########################################
:date: 2017-09-31 10:00
:tags: messaging, rabbitmq, qpid

OpenStack services make use of a message bus system for both remote procedure
calls (RPC) between components and to emit notifications. The aim of this spec
is to layout a plan for providing an alternative to RabbitMQ for RPC messaging.

https://blueprints.launchpad.net/openstack-ansible/+spec/hybrid-messaging

Problem description
===================

RabbitMQ is currently used as the message bus system for all remote procedure
calls (RPC) and notifications of OpenStack services deployed by
OpenStack-Ansible. While RabbitMQ is well tested and has wide acceptance across
OpenStack projects and deployments, it may not be the most efficient option for
RPC messaging. A brokerless message queue may provide greater performance of
messaging throughput and be less of a bottleneck, particularly in larger scale
deployments.

Proposed change
===============

This spec proposes offering Qpid Dispatch Router as an alternative option
for RPC messaging within an OpenStack-Ansible deployment.

Deployers will be able be given more options for messaging backends:

* RabbitMQ for both RPC and notifications (will remain the default deployment)
* Qpid Dispatch Router for RPC (with no dedicated backend for notifications)
* Qpid Dispatch Router for RPC and RabbitMQ for notifications (hybrid
  messaging)

Alternatives
------------

Leave RabbitMQ as the sole option for messaging within OpenStack-Ansible
deployments.

Playbook/Role impact
--------------------

Playbooks that deploy OpenStack services will need to be modified to make
any required against the deployer's messaging backend of choice.  Roles will
need to include additional package dependencies to connect to the Qpid
Dispatch Router.

Upgrade impact
--------------

An upgrade scenario will test the migration of a deployment from using
RabbitMQ.

Security impact
---------------

The default deployment of Qpid Dispatch Router should provide as close as
possible parity with OpenStack-Ansible's default RabbitMQ deployment including
use of TLS/SSL encryption and virtualhost namespacing of messaging data.

Performance impact
------------------

Especially in larger scale deployments, there is a potential improvement in
the throughput of messages and lowered CPU utilization.

End user impact
---------------

When chosen to be implemented by a deployer, the changes involved should be
transparent to end users.

Deployer impact
---------------

There would be no immediate impact to deployers as the changes involved would
be entirely opt-in initially. For deployers choosing to deploy Qpid Dispatch
Router, the service will be installed, likely in a new container, and OpenStack
services will be configured to make use of it.

Developer impact
----------------

New roles for OpenStack projects should include configuration options to allow
for using either RabbitMQ or Qpid Dispatch Router and testing of each.

Dependencies
------------

N/A

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  jimmy-mccrory (jmccrory)

Work items
----------

* Create a new role for the installation of Qpid Dispatch Router
* Create a playbook to deploy Qpid Dispatch Router
* Modify OpenStack service configuration templates within each role to allow
  a transport URL other than RabbitMQ and default variables to support that
* Add required client package dependencies to roles
* Create test scenarios in the roles to deploy using Qpid Dispatch Router as
  the messaging backend for RPC
* Create a common playbook for any Qpid Dispatch Router configuration changes
  required by individual OpenStack projects that the OpenStack project
  playbooks will consume
* Create test scenarios in the integrated gate for greenfield and upgrade
  deployments

Testing
=======

A Qpid Dispatch Router scenario would be created within the roles of OpenStack
projects which make use of a message queue and the integrated OpenStack-Ansible
repo to ensure installations and deployments, including upgrades, remain
functional.

Documentation impact
====================

Documentation will need to be added for the configuration options of Qpid
services, the configuration options for OpenStack services to make use of Qpid
services, and any associated maintenance tasks within the Operations Guide.

References
==========

AMQP 1.O (Qpid Dispatch Router) Oslo Messaging Driver Reference:

* https://docs.openstack.org/oslo.messaging/latest/admin/AMQP1.0.html

Message Routing- A Next-Generation Alternative to RabbitMQ:

* https://www.youtube.com/watch?v=R0fwHr8XC1I

Hybrid Messaging Solutions for Large Scale OpenStack Deployments:

* https://www.youtube.com/watch?v=o30YaqfLV9A
