Add support for multiple RabbitMQ clusters
##########################################
:date: 2016-07-11 21:00
:tags: rabbitmq, messaging, notifications, scalability

Larger deployments may wish to provision multiple RabbitMQ
clusters such that each cluster is deployed on its own set of hosts.

Such functionality would allow a deployer to configure one or
more additional component and container skeletons to add inventory
groups to be used for the deployment of additional clusters.

Problem description
===================

The current playbook and roles assume a single inventory group:
``rabbitmq_all`` that is deployed on the ``shared-infra_hosts``
infrastructure. The inventory group name is hardcoded throughout
and the playbook makes the assumption that only one cluster will
ever be needed.


Proposed change
===============

 * Modify the rabbitmq_server role to be more configurable with
   respect to the inventory group(s) that it operates upon
 * Modify the rabbitmq-install play to be more configurable with
   respect to the inventory group it operates upon

Alternatives
------------

I'm not aware of alternative ways for the project to address this need.


Playbook/Role impact
--------------------

Initial impacts will be to the rabbitmq_server role and the rabbitmq-install
play. However, I expect that additional impacts may exist within other roles
such that they would need to change to be more configurable with respect to
the inventory group they expect to use for rabbit hosts, or the variables they
use to identify which rabbit hosts they should connect to.


Upgrade impact
--------------

Unclear on how upgrades would be impacted. To my knowledge, custom inventory
extensions are not currently handled in the upgrade automation.


Security impact
---------------

No unique security impacts. The existing RabbitMQ security posture will
be maintained, though additional secrets may be required.


Performance impact
------------------

None expected/anticipated.


End user impact
---------------

End users will have increased flexibility in defining their deployment
architecture.


Deployer impact
---------------

The goal is for the deployer impact to be negligible due to the opt-in
nature of the changes discussed.


Developer impact
----------------

This change will add some additional complexity for developers, but it
should be minimal.


Dependencies
------------

None


Implementation
==============

Assignee(s)
-----------

Primary assignee:
  travis-truman (automagically)

Work items
----------

 * rabbitmq_server role modifications for inventory group configurability
 * rabbitmq-install play modifications for inventory group configurability
 * Documentation explaining how to create additional RabbitMQ cluster groups
 * Other role modifications to support cluster connectivity configurability


Testing
=======

This should be able to be tested within the rabbitmq_server role functional
tests given some changes to the test inventory.

Documentation impact
====================

An appendix should be added that explains to deployers how to configure
their environment for RabbitMQ multiple cluster support.

References
==========

None
