Octavia
#######
:date: 2016-11-01 00:00:00
:tags: lbaas, octavia, load balancer, neutron

Blueprint: Deploy Octavia (LBaaS) with OpenStack-Ansible
  Link: https://blueprints.launchpad.net/openstack-ansible/+spec/octavia

The `Octavia <https://wiki.openstack.org/wiki/Octavia>`_ project deploys load
balancers that are more scalable and resilient than the original neutron-lbaas
agent-based load balancers.  Octavia has a few daemons that handle the build-
out, configuration, and tear-down of load balancers.

Problem description
===================

There are two main load balancer offerings in OpenStack right now:

* LBaaSv2 w/agent: Uses the neutron-lbaasv2 agent with haproxy running in a
  namespace
* LBaaSv2 w/Octavia: Deploys load balancers into virtual machines and manages
  them using the LBaaSv2 API

Agent-based load balancers have scalability and reliability limitations since
the haproxy instances only run in one place without failover.

Octavia offers some helpful improvements for load balancing:

* Load balancers are deployed into virtual machines, which allows them to be
  sized appropriately and segregates them from the control plane.
* Putting load balancers into the virtual machines brings them closer to the
  resources that they are balancing. This increases load balancer performance,
  especially in clouds where the control plane is deployed on weaker hardware
  than the data plane (hypervisors).
* Octavia can deploy load balancers in a highly available configuration
  (currently active/passive) which helps with failures as well as
  patching/updates.

Proposed change
===============

The proposed changes would include:

* Create a role for Octavia (possibly `openstack-ansible-os_octavia``)
* Add Ansible code to deploy Octavia within an OpenStack-Ansible environment
* Add centralized tests for the Ansible role
* Add documentation to the role itself
* Integrate the role with OpenStack-Ansible without dislodging the existing
  neutron-lbaasv2 + agent support
* Allow deployers to choose LBaaSv2+agent or LBaaSv2+Octavia
* Add documentation to OpenStack-Ansible's main docs to explain how to deploy
  Octavia as part of the integrated build

Optionally, work could be done to enable SSL offloading support, which requires
a deployment of Barbican.

Alternatives
------------

We could keep using LBaaSv2 with the agent architecture until that code is
deprecated.  This is not ideal.

Playbook/Role impact
--------------------

Playbooks will need to be added to OpenStack-Ansible to deploy Octavia, but
this would be very similar to the existing work done for other services, like
Neutron.

Upgrade impact
--------------

Octavia hasn't been deployed previously, so there's nothing to upgrade here.
However, deployers who are currently using LBaaSv2+agent will have the option
of changing the backend LBaaSv2 driver to use Octavia instead. They will need
to delete all existing load balancers prior to making this change and recreate
them.

Security impact
---------------

The main security concern is that the Octavia load balancer virtual machines
will need to be on some type of management network that can be reached by
Octavia services that are running within the control plane. Those virtual
machines will have one network connection into a tenant network and one
connection into a management network.

This could allow an attacker to move from a compromised load balancer VM into
the control plane.  We will need to determine some ways to mitigate those types
of attacks.  This could be done with iptables or other network filtering.

Performance impact
------------------

Load balancing performance should be better with Octavia-based load balancers.
However, we will need to generate or download a VM image for the load balancer
virtual machines.  This could take time and it will need to be optimized.

End user impact
---------------

End users that already use the LBaaSv2 API won't notice a change.  The API
contract and endpoints remain the same. Only the backend LBaaSv2 driver will
be changed.

Deployer impact
---------------

Deployers will need to enable Octavia deployments if they choose to use them.
Octavia will not be deployed by default.  Deployers will also need to do their
capacity planning a little differently since load balancer virtual machines
will take up space within the data plane that would normally be occupied by
tenant virtual machines.

Developer impact
----------------

The new Octavia role will follow the same deployment/testing patterns as other
roles. It should be just as approachable as other OpenStack-Ansible independent
roles.

Dependencies
------------

The work for the Octavia role has no dependencies that are unsatisfied.

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  Major Hayden (IRC: mhayden)

Work items
----------

See the **Proposed change** section above for an itemized list.

Testing
=======

The Octavia role should use the standard centralized testing repository as
other roles. Octavia will need keystone, nova, neutron, glance deployed for
proper testing.

Barbican will be required for SSL offloading if that feature is enabled.

Documentation impact
====================

Documentation will be needed for the role itself, as well as in the integrated
repository. This documentation should match up with the docs written for other
services, like neutron or nova.

References
==========

Octavia wiki: https://wiki.openstack.org/wiki/Octavia
Octavia roadmap: https://wiki.openstack.org/wiki/Octavia/Roadmap
