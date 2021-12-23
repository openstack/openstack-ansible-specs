Enabling TLS on Internal Communications
#######################################
:date: 2022-11-16 21:00
:tags: ssl, tls, certificates, https, security

To improve the security of an OpenStack-Ansible deployments all traffic,
both internal and external should be encrypted. There is already
support for encrypting external traffic from all public endpoints that
reside behind haproxy, but this is not the case for all internal traffic.


Problem description
===================

This problem can broadly be split into 3 sections:

* Securing internal communications to the internal haproxy VIP

* Securing internal communications from haproxy to backends

* Securing internal communications between services such as rabbitmq, galera,
  nova live migration and noVNC


Securing internal communications to the internal haproxy VIP
------------------------------------------------------------

Support for using TLS on in the internal haproxy VIP is already present in
haproxy role and is enabled for the AIO deployment, but not enabled for new or
upgrades of existing deployments.

There are no issues with enabling TLS on the internal haproxy VIP for new
deployments, but for existing deployments an upgrade process needs to be
implemented. The reason an upgrade process is required is because currently
if you enabled TLS on the internal haproxy VIP it would cause downtime, until
each client is configured to use HTTPS instead of HTTP.

Problems to resolve:

* Haproxy configuration to allow TLS to be enabled without downtime of API's on
  existing deployments

* OpenStack-Ansible upgrade process and upgrade scripts to enable TLS without
  downtime of API's on existing deployments


Securing internal communications from haproxy to backends
---------------------------------------------------------

Securing the communications from haproxy to the services backends is as
important as securing communication to the internal haproxy VIP.

A large number of the services used with haproxy use UWSGI, meaning once TLS
support is added to the UWSGI role there is only configuration to enable TLS
and the generation of certificates required for each of the services.

For services that do not use USWGI, such a noVNC Proxy further investigation is
required.

As with enabling TLS on the internal haproxy VIP for new deployments, there is
no issue with enabling TLS from haproxy to backends, but an upgrade process for
existing deployments is required. The reason an upgrade process is required is
because if haproxy expects TLS backends, but TLS has not been enabled on the
service yet the connection will fail and if you enable TLS on the service the
connection will fail as haproxy is not configured for TLS.

Problems to resolve:

* Add TLS support to UWSGI

* Add configuration to role for each service that use UWSGI to enable TLS

* Add configuration to role for remaining services that do not use UWSGI

* Add configuration to OpenStack-Ansible to enable TLS on backend of each
  service

* OpenStack-Ansible upgrade process and upgrade scripts to enable TLS on
  backends without downtime of API's on existing deployments

Securing internal communications between services
-------------------------------------------------

Many OpenStack services communicate directly with each other and do not use
haproxy, these communications should also be secured. The work to secure these
communications is already complete and enabled in the Yoga release of
OpenStack-Ansible, for the following services:

* RabbitMQ

* Galera

* Nova live migrations

* noVNC (noVNC to compute nodes).

Problems to resolve:

* Secure the following services:

  - Memcached

  - etcd

  - OVN/OVS

* Are there any services missing from the list that do not go via haproxy that
  need their communications securing?

Proposed change
===============

Enable TLS on all internal communications.

Internal communications could be encrypted using a self-signed certificate,
but as OpenStack-Ansible has support for issuing certificates from a
self-signed private certificate authority using the ansible-role-pki, this
should be used instead as it both encrypts the data and allows a client to
trust the server.

In all cases a user should be able to override the certificates issued by a
self-signed private certificate authority, allowing them to provide their own
certificate which may have been issued by a publicly trusted certificate
authority.


Alternatives
------------

None, internal communications should be protected and TLS is an appropriate
and well used solution.


Playbook/Role impact
--------------------

Roles:

* Support for generating certificates using the ansible-role-pki role will be
  added to each service

* Configuring to enable/disable TLS will be added


Upgrade impact
--------------

Enabling TLS could be performed during or post upgrade.

As discussed in the problem description section, enabling TLS on the internal
haproxy VIP and service backends for existing deployments will cause downtime
during an upgrade if enabled. The reason it will cause downtime is that for both
communications from internal client => internal haproxy VIP (server) and
haproxy (client) => openstack service backend (server), both the client and
server need to be updated to use TLS at the same time.

To mitigate this issue I propose an intermediate step during an upgrade, where
haproxy frontend will accept both HTTP and HTTPS communications.
This would be achieved by adding a new TCP frontend to haproxy that accepts
both HTTP and HTTPS traffic and redirects to correct frontend for each,
and means that openstack clients can carry on using the same well known port
and haproxy looks after redirecting them to the correct frontend; HTTP or HTTPS.

To mitigate issues with haproxy<>backend communication, I suggest implementing
"Separated Haproxy Service Config" feature[1] that configures openstack service
and its haproxy service in the same playbook.

The other issue to be aware of is that when user wants to use predefined
certificate, this certificate will be used on all VIPs, both internal and
external.
This means that if TLS is enabled on haproxy's internal VIP, internal clients
must be able to trust the presented certificate if it is the same as the
external certificate.
This limitation does not apply to:
- certbot, which can present a separate certificate on external interfaces.
- PKI role which installs different certificates for external and internal
VIPs by default


Security impact
---------------

This change will encrypt all internal communications, securing any sensitive
data being sent, therefore security is improved.


Performance impact
------------------

Implementing TLS on all internal communications will lead to a small increase
in the processing requirements and latency of servers and clients, but the
increased security outweighs these.


End user impact
---------------

None, if the deployment is done correctly.


Deployer impact
---------------

* Deployer's will need to add monitoring of certificate expiry dates and renew
  is necessary, if a certificates expires connections between services will be
  dropped.

* This change should have no impact to deployer's of new deployments,
  OpenStack-Ansible will create the certificates, deploy them and
  configure all services to use them.

* This change will impact existing deployments and an upgrade process will be
  implemented to help minimise and possibly prevent this.


Developer impact
----------------

No impact, other that traffic will be encrypted meaning tools like tcpdump
may provide less useful as they will not be able to the see the contents of
packets.


Dependencies
------------

None.


Implementation
==============

Assignee(s)
-----------

Primary assignee:
  Damian Dabrowski
  <damian@dabrowski.cloud>


Work items
----------

* Enable TLS support to UWSGI role

* Enable TLS backend support to haproxy role

* Add configuration to openstack services that use UWSGI to create TLS
  certificate and enable TLS on UWSGI

* Add configuration to remaining openstack services that do not use USWGI to
  enable TLS support

* Add configuration in OpenStack-Ansible to allow TLS for all service to be
  enabled on both the server and haproxy

* Update documentation on TLS configuration options

* Add documentation for upgrade procedure

* Add script to automate as much as possible of the upgrade


Testing
=======

These changes can be tested using the existing setup, but manual testing of
upgrade procedure will be required to make this is does not cause any downtime,
as the automated testing only confirms a working upgrade at the end.


Documentation impact
====================

As this change will add extra configuration options these will need to be
documented.

The upgrade procedure for existing deployments will also have be documented,
as if this functionality is not deployed correctly it may cause system
distribution.


References
==========

[1] https://specs.openstack.org/openstack/openstack-ansible-specs/specs/antelope/separated-haproxy-service-config.html
