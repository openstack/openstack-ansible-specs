Load Balancers v2 (LBaaSv2 & octavia)
#####################################
:date: 2016-01-28 00:00
:tags: lbaasv2, octavia, load balancing, neutron

Blueprint: Load Balancers v2 (LBaaSv2 & octavia)
  * https://blueprints.launchpad.net/openstack-ansible/+spec/lbaasv2

OpenStack-Ansible currently offers LBaaSv1, but it is deprecated in Liberty
and it is scheduled to be removed in Newton.  LBaaSv2 became stable in Liberty
and it provides several improvements over LBaaSv1:

* **Multiple TCP ports per load balancer:** This helps when hosting websites
  which must serve visitors on ports 80 and 443. It can also reduce the number
  of floating IP address that are required for deployment.
* **Failover support:** Deployers have the option to deploy a single load
  balancer node or an active/passive pair.
* **Load balancers inside guests:** The load balancer runs inside virtual
  machines rather than alongside other neutron agents.
* **Housekeeping:** If a load balancer goes offline or haproxy is down, the
  health manager will remove the faulty load balancer and build a new one in
  its place.
* **TLS Termination:** TLS can be terminated at the load balancer so that
  individual virtual machines aren't required to encrypt/decrypt traffic.

However, LBaaSv2 does have some limitations:

* **Horizon support:** Horizon support for LBaaSv2 is *planned* for Mitaka and
  work is currently in progress, but the panels are not yet available at the
  time of writing of this spec.
* **Cannot run alongside v1:** It is not possible for both versions of LBaaS to
  run at the same time, and there is not a migration path available today for
  users who want to migrate their v1 load balancers to v2.

Problem description
===================

Although LBaaSv1 support exists in OpenStack-Ansible already, it is deprecated
in Liberty and it is scheduled for removal in Newton.  It also has a limitation
of one listening port per load balancer, which limits a user's ability to host
an application on more than one port (think HTTP and HTTPS) on a single load
balancer.

LBaaSv2 replaces LBaaSv1 and will be supported by OpenStack developers going
forward.

Proposed change
===============

There will be several changes needed to deploy LBaaSv2:

#. **Add octavia to existing neutron virtualenv:** The octavia project will
   need to be included with the neutron virtualenv that is deployed within
   the neutron-server container.
#. **Manage four octavia daemons:** Four daemons will need to run in the
   neutron-server container: octavia-api, octavia-housekeeping, octavia-worker,
   and octavia-health-monitor.
#. **Deploy octavia's configuration file:** The ``/etc/octavia/octavia.conf``
   will need to be deployed and managed.
#. **RabbitMQ/MariaDB credentials:** Octavia will require its own database,
   database credentials, and RabbitMQ credentials.
#. **Neutron configuration changes:** Changes will be needed in
   ``neutron.conf`` to add the LBaaSv2 service plugin and driver.
#. **Update documentation:** Deployers would need to know the difference
   between both load balancer implementations and how to deploy each. Deployers
   using LBaaSv1 would also need some advice on how to handle a change to
   LBaaSv2 in their OpenStack environments.

Alternatives
------------

We could choose to stay with v1 through the Mitaka release, but then we might
be forced to remove it for the Newton release in favor of v2.

Playbook/Role impact
--------------------

The vast majority of the changes should take place in the neutron role. LBaaSv1
is currently enabled by adding a service_plugin entry, and the same could be
done for LBaaSv2.  When the LBaaSv2 service_plugin entry is present, the
neutron role would ensure that octavia is running and ready to receive
requests.

Upgrade impact
--------------

The upgrade impact depends on whether a deployer is currently using LBaaSv1.

If a deployer is already using LBaaSv1, they would need to carefully consider
their migration path to v2 since both implementations cannot run concurrently.
However, if a deployer is already using v1, they could upgrade to Liberty or
Mitaka without making any adjustments to how they use load balancers.
Upgrading to Newton would require changes since LBaaSv1 is expected to be
removed in that release.

If a deployer is not using LBaaSv1 at this time, then they would simply be
gaining functionality that they did not have previously when they upgrade to
Liberty, Mitaka, or Newton.

Security impact
---------------

There are no sigificant performance impacts within LBaaSv2, but it could
allow deployers to deploy HTTPS websites on the same IP address as their HTTP
site.

Octavia also integrates with the Anchor and Barbican projects. Anchor allows
users to obtain certificates from a pre-configured certificate authority (CA)
within their OpenStack environment and Barbican can be used to store private
keys for SSL/TLS connections.

Performance impact
------------------

LBaaSv2 should scale better than LBaaSv1 since it runs within virtual machines
rather than inside the neutron-agents container with multiple haproxy-agents.

End user impact
---------------

If an end user is not currently using LBaaSv1, then they would be gaining a new
feature that they could consume via an API.  Horizon panels are planned for
Mitaka.

If an end user is currently using LBaaSv1, they would lose load balancer
functionality when LBaaSv2 is deployed until they configure their load
balancers within the LBaaSv2 API.  Deployers must work closely with end users
to determine the best path forward.

Deployer impact
---------------

If a deployer is not currently using LBaaSv1, they could easily enable LBaaSv2
by adding a service_plugin within their ``openstack_user_config.yml`` for
LBaaSv2, just as they do for LBaaSv1 today.

If a deployer is currently using LBaaSv1, they could continue using it without
interruption (until the Newton release).  If they choose to move to LBaaSv2,
they would need to coordinate the changes with their end users to avoid lengthy
service interruptions.

Developer impact
----------------

The LBaaSv2 deployment would be part of the neutron deployment, much like
the current deployment process for LBaaSv1.  No additional roles or playbooks
are required.

Dependencies
------------

This spec does not depend on any other development work in OpenStack-Ansible.

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  Major Hayden (IRC: mhayden, LP: rackerhacker)

Other contributors:
  None

Work items
----------

See the details in the *Proposed Changes* section above.

Testing
=======

Tempest testing exists for the LBaaSv2 API but tempest tests for the octavia
API are still in progress.

Documentation impact
====================

Some topics are mentioned above in the *Proposed Changes* section. The
following topics must be documented:

* What is different between LBaaSv1/2?
* What do I do if I already deployed LBaaSv1?
* How do I migrate from v1 to v2?
* How do I deploy/configure LBaaSv2?
* How do I troubleshoot LBaaSv2 issues?

References
==========

Mailing list discussion:

* `LBaaSv2 / Octavia support`_

Software:

* https://github.com/openstack/octavia
* https://github.com/openstack/neutron-lbaas

Documentation:

* `LBaaSv2 in Devstack`_
* `Load Balancing as a Service v2.0 - Liberty and Beyond (PDF)`_
* `Networking API v2.0 extensions`_

.. _LBaaSv2 in Devstack: http://docs.openstack.org/developer/devstack/guides/devstack-with-lbaas-v2.html
.. _Load Balancing as a Service v2.0 - Liberty and Beyond (PDF): https://www.openstack.org/assets/Uploads/LBaaS.v2.Liberty.and.Beyond.pdf
.. _Networking API v2.0 extensions: http://developer.openstack.org/api-ref-networking-v2-ext.html
.. _LBaaSv2 / Octavia support: http://lists.openstack.org/pipermail/openstack-dev/2016-January/085022.html
