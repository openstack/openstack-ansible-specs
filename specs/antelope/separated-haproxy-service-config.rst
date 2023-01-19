Separated Haproxy Service Config
#################################
:date: 2023-01-19 22:00
:tags: separated haproxy service config, internal tls

Currently, all haproxy services are configured during execution of
haproxy-install.yml playbook.
It may cause issues with variables scope or even completely break a service
until service role is executed.

These issues may be avoided if the current behavior will be changed and
haproxy services will be configured separately at the beginning of each
service playbook.


Problem description
===================

Preconfiguring all haproxy services may lead to some issues.
There are 2 examples:

1. Variables scope

   Currently, haproxy service definitions stored in
   inventory/group_vars/haproxy/haproxy.yml refer to variables like
   ``neutron_plugin_type``.
   It makes a problem because this is neutron's variable.
   If someone wants to change its value, they will probably set an override in
   neutron group variables. It's problematic because haproxy is not in neutron
   group, so all neutron's variables won't have an effect for haproxy role.
   In order to make haproxy respect this change, variable needs to be defined for
   all hosts, so both haproxy and neutron will have access to it.

   Additionally, we are currently working on encrypting traffic between haproxy
   and service backends. Proposed PoC [1] does the same thing as described above.
   It refers to ``glance_backend_https`` variable belonging to glance role.

2. Strong dependency between haproxy role and service roles

   Some changes in haproxy service need an immediate reaction from service role.
   For example: user enables TLS for communication between haproxy and glance
   backends. To fix that, haproxy role needs to be executed.
   It will configure glance service to communicate with its backends over TLS,
   but at this point backends are not ready to handle TLS connections.
   To fix it, glance role needs to be executed, but it takes time and
   increases downtime. Removing dependencies like this between roles would make
   the configuration process more reliable.

   Please note that downtime will still occur. It will start after haproxy service
   config step and finish after first backend host will be configured.
   To provide zero-downtime transition to TLS, further work related to
   "internal TLS" project is required.


Proposed change
===============

Add an extra step at the beginning of each service role to configure haproxy
service(s) for it.
In this case, haproxy services will be configured separately, so nova playbook
will configure nova haproxy services, glance playbook will configure its own
haproxy services etc.
Haproxy playbook will be only responsible for configuring services not related
to any openstack role(letsencrypt, ceph-rgw, custom user-defined services etc.)


Alternatives
------------

No alternatives.


Playbook/Role impact
--------------------

Each playbook will contain an extra step responsible for configuring haproxy
service role(s) for it.


Upgrade impact
--------------

Some variables will be removed or replaced. It's already covered in release
notes.

Security impact
---------------

No impact.


Performance impact
------------------

No impact.


End user impact
---------------

No impact.


Deployer impact
---------------

From now on, haproxy services will be configured separately when running
service playbooks(like os-nova-install.yml)

Developer impact
----------------

No impact.

Dependencies
------------

No dependencies.


Implementation
==============

Assignee(s)
-----------

Primary assignee:
  Damian Dabrowski
  <damian@dabrowski.cloud>

Work items
----------

- configure haproxy_server role to support separated service config
- configure service playbooks to include an extra step to configure haproxy
  service
- solve all corner cases(like dependency between letsencrypt and horizon)


Changes Hierarchy
-----------------

All changes are available on gerrit under 'separated-haproxy-service-config' tag[2].
It may be hard to understand relations between them so here is a description
Changes at the top should be merged first.
Horizontal lines split dependency groups(changes in the same group may be merged independently)


* Blueprint for separated haproxy service config
  https://review.opendev.org/c/openstack/openstack-ansible-specs/+/871187
* [openstack-ansible] Define some temporary vars for haproxy
  https://review.opendev.org/c/openstack/openstack-ansible/+/872328

----

* [haproxy_server] Prepare haproxy role for separated haproxy config
  https://review.opendev.org/c/openstack/openstack-ansible-haproxy_server/+/871188

----

* [openstack-ansible] Prepare service roles for separated haproxy config
  https://review.opendev.org/c/openstack/openstack-ansible/+/871189

----

* [haproxy_server] [DNM] Remove temporary tweaks related to separated haproxy service config
  https://review.opendev.org/c/openstack/openstack-ansible-haproxy_server/+/871194


Testing
=======

Special attention is required for gating. Merging this change for all
roles may be complicated.


Documentation impact
====================

Documentation needs to updated in a few places. Uploaded changes already
contain these updates.


References
==========

[1] https://review.opendev.org/c/openstack/openstack-ansible/+/821090

[2] https://review.opendev.org/q/topic:separated-haproxy-service-config
