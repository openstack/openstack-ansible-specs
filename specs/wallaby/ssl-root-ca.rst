SSL Root Certificate Authority
##############################
:date: 2020-10-19 14:00
:tags: ssl, haproxy, nova, galera, infra

Blueprint on Launchpad:
  * https://blueprints.launchpad.net/openstack-ansible/+spec/ssl-root-ca

Having SSL encryption is a vital part of every service in the modern world.
OpenStack-Ansible already provides deployers options how to cover public
and internal endpoints with SSL certificates and allows the generation and
use self-signed certificates. However we can make these certificates
"verified" by usage of the root CA, which will be distributed across all
containers and services. This will increase security as users will be
alerted in case of the certificate verification failure, since certificate
verification will be enabled.

Problem description
===================

At the moment several openstack-ansible roles do create self-signed
certificates their own way which does not provide any consistency and is
pretty hard to maintain. Moreover, these certificates are self-signed
ones, which means that certificate verification has to be disabled for such
certificates. So in case of the certificate "impersonation" you won't be
alerted, and encrypted data might be not secure anymore.

Futhermore, for mysql SSL usage, it is required to place Root CA on
the client containers for mysqlclient to reach the database,
while at the moment we have only server side encryption covered,
but Root CA distribution remains the responsibility of the deployer.
Another good example is nova, where in order to disable tunneling for the
live migrations and to use block device migrations, we should be securing
the connection with mutually verifiable certificates.

To resolve all of the problems above we need:

  * Root CA certificate (and corresponding key which may not be available)
  * An intermediate signing certificate and key

Proposed change
===============

In order to resolve issue we need to create a root certificate authority on
the deploy host, which will be used for the futher creation of the
certificates.

We need this for (but not limited to):
  * Creating a self-signed certificate for HAProxy in CI
  * Creating ssh signed certs to replace ssh keys in nova and keystone roles
  * To use TLS for live migration
  * To use TLS for galera and other infrastructure services
  * To use TLS for connection between HAProxy and uwsgi

Implementation
--------------

Create role in separate repo which would consist from several parts
which would be included wherever needed:

  # Create/rotate or verify existing provided CA on the deploy host.
  Included in openstack_hosts, to distribute CA to certificate storage
  We should implement proper Root CA rotation mechanism including usage of the
  OldWithOld, OldWithNew, and NewWithOld.
  # Create/rotate or verify existing key and certificate, which would be also stored
  on the deploy host. Will be included in required roles during their runtime
  Each instance of each component uses a unique certificate.
  # Decide what we 'call' the internal VIP if it's needed to verify the dns
  name against subject name in an SSL certificate.
  Consider having support for SAN certificates that will include several
  names or IPs.
  # Create a signed ssh key which can be validated against the CA certificate
  to avoid needing to distribute all keys to all hosts.
  # Role should have a specific key to rotate self-signed certificates and
  root CA when it's asked to do so.
  It's up to deployer to keep track on the certificates exipration date. Since
  they are placed on the deploy host, it should be pretty straightforward to
  configure monitoring tool for that.

Roles/service impact
====================

For all hosts/containers
------------------------

The Root CA is installed into the system trust store
Consider pointing REQUESTS_CA_BUNDLE to the system trust store rather than
certificate bundle.

For HAproxy
-----------

We will have the following user scenarios:
  * The user supplies their own cert and key and points variables to the files
  * Letsencrypt creates the certificate
  * A self signed certificate and key is created at deploy time by the OSA role
  * Disable SSL certificate usage

.. note::

    Self signed cert is required to bootstrap LE for the first run

For Galera (or other infrastructure service)
--------------------------------------------

We will have the following user scenarios:

  * The user supplies their own cert and key and points variables to the files

  * A self signed certificate and key is created on the deploy host at deploy
    time by the OSA role (stored in a well known location on the deploy host).
    The existing ansible roles pick these up

  * Disable SSL certificate usage


For service components such as Nova and Octavia which can use TLS
-----------------------------------------------------------------

We will have the following user scenarios:

  * The user supplies their own cert and key and points variables to the files

  * A self signed certificate and key is created on the deploy host at deploy
    time by the OSA role (stored in a well known location on the deploy host)
    The existing ansible roles pick these up

  * Disable SSL certificate usage

To replace ssh keys
-------------------

  * Signed ssh certificates are created on the deploy host and copied to the
    relevant .ssh user directories. The signing CA is installed into the
    relevant ssh_config file in /etc/
  * The deployer may already be using signed ssh keys for access to hosts so
    any implementation should work alongside existing configuration. It
    may be necessary to investigate supporting more than one trusted CA in
    the ssh_config file.


Upgrade impact
--------------

By default self-singed alternatives will be used for all types of services.
In case deployer would like to omit that, he will need to explicitly disable
that behaviour before upgrade.
No other impact for upgrades is planned at the moment.


Security impact
---------------

Realization of this blueprint should make systems more secure because
of the SSL usage for most of the interactions and enabling SSL verification
even for the self-singed sertificates.


Performance impact
------------------

This may decrease performance slightly because SSL encryption requires several
extra CPU cycles and TLS handshake to be performed, however this drawdown can
be neglected.


End user impact
---------------

No end user impact


Deployer impact
---------------

The deployer will be able to provide:
  * Root CA
  * An intermediate cert and key

Also we should leave possible to disable SSL usage for services.


Developer impact
----------------

This blueprint will ease maintenance of the roles, because all SSL-specific
parts will be moved to the standalone role. So in case of the need to change
specific task it would be done in a single place rather than in each role.



Implementation
==============

Assignee(s)
-----------


Primary assignee:
  noonedeadpunk

Other contributors:
  jrosser


Work items
----------

TBD


Testing
=======

We will enable self-singed certificates usage in CI


Documentation impact
====================

Documentation of the added options and architecture should be added at the
end of the day, as well as release notes.


References
==========

  * Etherpad discussion: https://etherpad.opendev.org/p/osa-certificates-refactor
  * Root CA Key Update https://tools.ietf.org/html/rfc4210#section-4.4