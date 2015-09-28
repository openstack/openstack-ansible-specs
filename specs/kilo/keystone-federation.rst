Keystone Federation
###################
:date: 2015-06-22 08:00
:tags: federation, scalability

This spec is to propose adding support for Keystone federation to
openstack-ansible.

Launchpad blueprint: https://blueprints.launchpad.net/openstack-ansible/+spec/keystone-federation

Operators of private clouds often have the need for additional capacity or
services found in other private and public clouds. OpenStack has accommodated
this use case through Keystone Identity Federation, allowing identity
credentials from one cloud to act as authorization in another.

The primary use case would be for a private cloud to act as an Identity
Provider (IdP) to other clouds, typically Public Clouds. This allows users
found in the private cloud databases to authenticate in order to consume
resources provided by other Service Provider (SP) clouds.

The secondary use case is where a private clouds would work as a SP to another
Private Cloud or external provider acting in the role of an IdP.

Problem description
===================

* As a User, in order to utilize my Keystone identity to consume resources in
  other Keystone backed Service Providers, I should be able to effectively
  authenticate with those Service Providers using only my Keystone identity
  credentials via the Service Provider's Command Line Interface (CLI).

* As an Administrator, in order to allow my users to utilize their Keystone
  identity with other Service Providers, I should be able to establish a trust
  relationship between my Keystone and a Service Provider Keystone via CLI.

* As an Administrator of multiple clouds, in order to provide identity
  federation between my multiple clouds, I should be able to establish a trust
  relationship between my Keystone Identity Provider Cloud and my Keystone
  Service Provider clouds.

* As an Administrator, in order to effectively map Identity Provider groups and
  users to Service Provider roles, I should be able to simply define mappings
  to Service Provider projects, domains and roles for given groups.

* As a Deployer, in order to prevent downtime or interruption, I should be able
  to setup my cloud as an Identity Provider or Service Provider with little or
  no interruption to the data plane.

* As a User, in order to understand the resources available to me, I should be
  able to retrieve a list of Service Providers which trust my Identity Provider
  as well as a service catalog for the services offered by those Service
  Providers.

* As an Administrator, in order to use Identity Federation between my secured
  network Private Cloud and other Public Service Providers, I should be able to
  easily establish a trust relationship between the two without compromising my
  network security.

Proposed change
===============

1. Enable and configure Keystone Federation, implementing the IdP/SP
   configuration in a manner that is simple for deployers and requires
   little or no data plane downtime. The initial SP configuration will use
   saml-based authentication and Apache mod_shib. Later options to extend
   support to would include the saml-based Apache mod_auth_mellon, the
   OpenID-based Apache mod_auth_openidc, the kerberos-based Apache
   mod_auth_kerb/mod_auth_identity.

2. Improve the configuration of Keystone SSL endpoints to ensure that both
   the IdP and SP public interfaces can be served via SSL using a supplied
   server key, server certificate, Certificate Authority certificate and
   (optionally) an intermediary certificate.

3. Change the Keystone and Utility containers to use the python-openstackclient
   instead of the python-keystoneclient in order to ensure that the Keystone
   v3 API may be used. This is required for the administration of Federation
   IdP and SP configuration entities.

4. Change the Horizon configuration to allow it to consume the Keystone v3 API.

5. Automate the registration of a trusted IdP to an SP.

6. Automate the registration of a list of trusted SP's to an IdP.

7. Document and, if possible, automate the registration and mapping of
   external identities to specified domains, projects, roles and users.

Alternatives
------------
None


Playbook/Role impact
--------------------
1. The os_keystone role will require changes to both tasks and templates in order
   to facilitate the configuration of the IdP, SP, openstackclient and SSL.

2. The os_horizon configuration will require changes to the templates in order to
   facilitate the change to use the Keystone v3 API.

3. The openstack_openrc role may need to be changed in order to place a different
   openrc file into the keystone and utility containers.

4. The automation of registration and mapping of external identities to specific
   domains, projects, roles and users may be done in a new playbook/role or within
   the existing keystone playbook/role.

5. The os_keystone role will need to include the capability to replicate the same
   SP signing certificates from the first Keystone container to all the others.

Upgrade impact
--------------
Horizon will be reconfigured to use the Keystone v3 API.


Security impact
---------------
1. Keystone must have its public endpoint implemented with SSL in order to encrypt
   and protect authentication and identity information which is communicated
   between the IdP and the SP.

2. The SP signing key and certificate must be properly secured on every server
   that has it.


Performance impact
------------------
TBD


End user impact
---------------
1. Administration of keystone via the v3 API will mean switching from using
   the 'keystone' CLI to using the 'openstack' CLI.

Deployer impact
---------------
1. The deployer will be able to implement specified SSL certificates for the
   Keystone public endpoints.


Developer impact
----------------
1. The keystone Ansible module will be updated to make use of the keystone
   v3 API.


Dependencies
------------
1. Keystone IdP requires the following:
   * xmlsec1: http://packages.ubuntu.com/search?keywords=xmlsec1
   * python-openstackclient: https://pypi.python.org/pypi/python-openstackclient

2. Keystone SP requires the following:
   * xmlsec1: http://packages.ubuntu.com/search?keywords=xmlsec1
   * libapache2-mod-shib2: http://packages.ubuntu.com/search?keywords=libapache2-mod-shib2
   * python-openstackclient: https://pypi.python.org/pypi/python-openstackclient

3. Keystone mapping documentation:
   * https://review.openstack.org/192850

4. Keystone SP must use uuid tokens for now
   * https://bugs.launchpad.net/keystone/+bug/1471289

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  https://launchpad.net/~miguelgrinberg (miguelgrinberg)

Other contributors:
  https://launchpad.net/~hughsaunders (hughsaunders)
  https://launchpad.net/~icordasc (sigmavirus24)
  https://launchpad.net/~jesse-pretorius (odyssey4me)


Work items
----------
1. Convert existing Keystone Ansible module to use v3 API

2. Add federation commands to Keystone Ansible Module

3. Keystone public endpoint SSL configuration

4. Keystone/Utility container implementation of python-openstackclient

5. Keystone IdP software deployment, configuration and SP registration

6. Keystone SP software deployment, configuration and IdP registration

7. Document and, if possible, automate the registration and mapping of
   external identities to specified domains, projects, roles and users.


Testing
=======

Due to the nature of this feature requiring two independant
installations there will be no specific gate testing for it.

All changes implemented in the roles/plays as a result of this work will
need to be done in such a way that the existing gate checks continue to
pass.


Documentation impact
====================

1. The upgrade impact will need to be noted in the release notes.

2. The method of implementing the required user_variables for an IdP/SP
   will need to be described.

3. The specifics of registering and mapping external identities to
   domains, projects, roles and users will need to be documented.

References
==========

* http://docs.openstack.org/developer/keystone/configure_federation.html

* http://docs.openstack.org/developer/keystone/extensions/federation.html

* http://docs.openstack.org/developer/keystone/extensions/shibboleth.html

* http://blog.rodrigods.com/it-is-time-to-play-with-keystone-to-keystone-federation-in-kilo/

* https://zenodo.org/record/11982/files/CERN_openlab_Luca_Tartarini.pdf

