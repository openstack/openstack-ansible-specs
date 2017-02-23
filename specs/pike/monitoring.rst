Monitoring for an OpenStack-Ansible deployment
##############################################
:date: 2017-02-21 00:00
:tags: monitoring, operations

Include the URL of your launchpad blueprint:
  * https://blueprints.launchpad.net/openstack-ansible/+spec/example

The goal of the efforts described in this spec is to provide an easy method for
monitoring an OpenStack cloud. This would initially include basic service state
monitoring with extra functionality added as it matures.

Problem description
===================

OpenStack clouds are complex systems of hardware, software, and networks.
Deployers need to monitor the health of all of these components to ensure that
end users have access to resources. OpenStack-Ansible does not offer any
components for monitoring at this time, and this forces deployers to build
their own monitoring plugins and tool stacks.

Deployers and operators need to know:

* Are my OpenStack services up or down?
* Are my additional services (Galera, RabbitMQ, etc) up or down?
* What is the state of cluster partitions for Galera and RabbitMQ?
* Are my APIs responding within a reasonable time period?
* Are my management and tenant networks accessible?
* Is the hardware underneath my cloud operating normally?

Proposed change
===============

The proposed changes fall into two main buckets:

Monitoring plugins
  This is the **primary** work effort for the spec.

  Deployers need a solid set of monitoring plugins that gather information from
  various services or entities, and those plugins should output data in common
  formats for the most popular monitoring tool stacks.

Monitoring tool stack
  This is the **secondary** work effort for the spec.

  There are many open source and commercially available monitoring tool stacks
  available for Linux. Deployers should have the option to deploy an
  opinionated tool stack via Ansible if they don't have one of their own
  already. The tool stack should offer up its time series data for searching
  and also have an alerting mechanism that can hook into a deployer's existing
  notification tools.

Alternatives
------------

There are loose collections of monitoring plugins available within OpenStack,
but those plugins aren't being actively maintained. Many of the other plugin
sets available today only output their data in a specific format. Deployers
could choose to use these plugins instead.

Deployers could also deploy their own monitoring tool stacks if needed. They
could use the monitoring plugins created in this spec with their existing
tools.

Playbook/Role impact
--------------------

The monitoring plugins should be installable via pip and they can be added into
existing roles or playbooks *(perhaps the openstack-ansible-openstack_hosts
role)*. The plugins themselves should be released independently of an
OpenStack release.

The monitoring tool stack would be implemented in a new role with an additional
playbook. The role that deploys this stack would be versioned along with
OpenStack-Ansible releases so that it can utilize the existing variables and
modules from each release.

Upgrade impact
--------------

This would be the first implementation of monitoring plugins and tools in
OpenStack-Ansible, so there is no upgrade concern at the moment. However, the
plugins and tool stack installation should be written such that upgrades are
reliable.

Security impact
---------------

Some monitoring plugins will need some level of privileged access to OpenStack
services or the other services running in the cloud. This requires accounts to
be created and new secrets to be stored. It is possible to use accounts that
have fewer privileges so that a compromise of a monitoring plugin would have
a limited security impact.

The monitoring tool stack itself has important security concerns to address.
Data from the monitoring plugins running on each host or container must be able
to reach a centralized database for storage and processing. Access to any
web frontends or databases should be handled carefully, just as we do for
Horizon or Galera today.

Performance impact
------------------

Some monitoring plugins will need to make requests to OpenStack APIs or access
certain other services. These plugins must be written carefully to avoid
negative performance impacts on the system.

End user impact
---------------

End users should not notice the changes from this work.

However, they should get a better user experience if the environment is closely
monitored and operations teams have access to valuable performance data.


Deployer impact
---------------

The monitoring plugins should be distributed as a pip package, so this should
have a small impact on deployers.  Some plugins will need accounts on the
system, so deployers will need to create additional secrets for those accounts.

Deployers would have the option to deploy the monitoring tool stack if they do
not have one of their own.

Developer impact
----------------

The developer impact from these changes is very low.  The monitoring plugins
should be easy to use and heavily tested.  Developers should be able to modify
existing plugins and create new ones with ease.

Dependencies
------------

There are no dependencies.

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  Major Hayden (mhayden)

Other contributors:
  Kevin Carter (cloudnull)
  Antony Messerli (antonym)

Work items
----------

1. Write a small class that can be extended for new monitoring plugins.
2. Begin writing monitoring plugins that are executable via setuptools entry
   points.
3. Ensure that tests are available for each plugin as well as the base class.
4. Create a role to deploy a monitoring tool stack that uses these plugins.
5. Document the plugins and the tool stack.

Testing
=======

The monitoring plugins should be tested on each commit using tox.

The monitoring tool stack role should be tested independently (like the other
IRR repos) and added to the integrated build as an optional component.

Documentation impact
====================

The plugins should be documented and there should be developer guides that
explain how to modify existing plugins or add new ones. The monitoring tool
stack role will need documentation that explains the new variables and
functionality available.

References
==========

Notes from the OpenStack PTG in Atlanta (Feb 2017):

* https://etherpad.openstack.org/p/osa-ptg-pike-monitoring
