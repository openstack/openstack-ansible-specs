Keystone Service Provider with ADFS Identity Provider Deployment
################################################################
:date: 2015-06-22 10:00
:tags: federation, scalability

This spec is to propose adding support to openstack-ansible for Keystone
federation using an Active Directory Federation Service (ADFS) Identity
Provider.

Launchpad blueprint: https://blueprints.launchpad.net/openstack-ansible/+spec/keystone-sp-adfs-idp

OpenStack cloud deployers frequently utilize Microsoft Active Directory (AD)
as a corporate identity provider. In this case, provisioning user credentials
specifically for their OpenStack clouds, and managing/updating the
corresponding permissions for those users is burdensome. Deployers would
rather use Keystone's Federation capabilities with ADFS to have AD act as an
Identity Provider (IdP) to Keystone as a Service Provider (SP).

Problem description
===================

* As a User, in order to utilise my AD identity to consume resources in my
  OpenStack Cloud, I should be able to authenticate to my OpenStack Cloud
  using my AD credentials via the Service Provider's Horizon Dashboard and
  Command Line Interface (CLI).

* As an Administrator, in order to maintain one identity system, I should be
  able to create a trust relationship between my ADFS IdP and my OpenStack
  SP's.

* As an Administrator, in order to effectively map Identity Provider groups and
  users to Service Provider roles, I should be able to simply define mappings
  to Service Provider projects, domains and roles for given groups.


Proposed change
===============

1. Enable and configure Keystone as a federated SP with SSL public endpoints.
   The initial SP configuration will use saml-based Apache mod_shib. Later
   options to extend support to would include Apache mod_auth_mellon.

2. Document the configuration of the ADFS IdP in order to support the
   Keystone SP.

3. Change the Horizon configuration to support Web Single-Sign-On (SSO),
   thereby providing support for end-users to authenticate using their AD
   credentials.

5. Automate the registration of the trusted ADFS IdP to the Keystone SP.

7. Document and, if possible, automate the registration and mapping of
   external identities to specified domains, projects, roles and users.


Alternatives
------------
None


Playbook impact
---------------
1. The os_horizon configuration will require changes to the templates in order to
   facilitate the change to use WebSSO.


Upgrade impact
--------------
None


Security impact
---------------
There are security aspects, but they affect docs more than code:

* Security is to some extent delegated to the external IDP (AD). Therefore
  Deployers must be confident of the security of their AD before using it for
  federation.
* Deployers must take time to understand the mapping mechanisms in order to
  ensure that only the expected users/groups are granted access to OpenStack
  resources.


Performance impact
------------------
TBD


End user impact
---------------
1. If an external IdP is configured, Horizon will show multiple methods of
   authentication available via a drop-down list. The user will be able to
   choose between 'credentials' and the available WebSSO sources.


Deployer impact
---------------
None


Developer impact
----------------
None


Dependencies
------------
1. Keystone Federation Deployment Implementation:
   * https://blueprints.launchpad.net/openstack-ansible/+spec/keystone-federation
2. Horizon requires the following:
   * django-openstack-auth v1.2.0 or higher: https://pypi.python.org/pypi/django_openstack_auth


Implementation
==============

Assignee(s)
-----------

Primary assignee:
  https://launchpad.net/~jesse-pretorius (odyssey4me)

Other contributors:
  https://launchpad.net/~hughsaunders (hughsaunders)
  https://launchpad.net/~icordasc (sigmavirus24)
  https://launchpad.net/~miguelgrinberg (miguelgrinberg)


Work items
----------

1. Add the required ADFS configuration to the Keystone SP.
   * shibboleth2.xml
   * attribute-map.xml

2. Document the configuration of the ADFS IdP in order to support the
   Keystone SP.

3. Automate the registration of the trusted ADFS IdP to the Keystone SP.

4. Change the Horizon configuration to support Web Single-Sign-On (SSO),
   thereby providing support for end-users to authenticate using their AD
   credentials.

5. Document and, if possible, automate the registration and mapping of
   external identities to specified domains, projects, roles and users.


Testing
=======

Due to the nature of this feature requiring an installation of ADFS
(which is not possible in OpenStack-CI) there will be no specific gate
testing for it.

All changes implemented in the roles/plays as a result of this work will
need to be done in such a way that the existing gate checks continue to
pass.

Documentation impact
====================

1. The preparation of the ADFS IdP to support the Keystone SP will need
   to be described.

2. The method of implementing the required user_variables for the Keystone
   SP will need to be described.

3. The specifics of registering and mapping external identities to
   domains, projects, roles and users will need to be documented.

References
==========

* http://docs.openstack.org/developer/keystone/extensions/websso.html

* http://specs.openstack.org/openstack/keystone-specs/specs/kilo/websso-portal.html

* https://wiki.shibboleth.net/confluence/display/SHIB2/NativeSPADFS

* https://zenodo.org/record/11982/files/CERN_openlab_Luca_Tartarini.pdf
