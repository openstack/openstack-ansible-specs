Additional Role for Gnocchi Deployment
########################################
:date: 2016-01-20 11:20

:tags: gnocchi, openstack-ansible

The purpose of this spec is to add support for the OpenStack Gnocchi program
to OpenStack-Ansible. This would allow the deployment of Gnocchi along with
the core OpenStack components using OpenStack-Ansible.

Blueprint - Gnocchi deployment on OpenStack-Ansible:

https://blueprints.launchpad.net/openstack-ansible/+spec/role-gnocchi


Problem description
===================

Presently, while deploying OpenStack using OpenStack-Ansible only the core
OpenStack components get deployed. The deployment of other components
(eg: Gnocchi) via Ansible playbooks is not supported yet and to use this
component's services, they need to be deployed manually.

Gnocchi[1] is a multi-tenant timeseries, metrics, and resources database. It
is designed to store metrics at a very large scale and to allow the retrieval
of those metrics quickly and efficiently, each through an HTTP REST interface.
Additionally, Gnocchi is designed to stand as a replacement storage engine for
metrics processed through Ceilometer, relying on a more performant storage
format.


Proposed change
===============

This spec proposes to allow deployment and management of this service with a
versatile configuration capable of scaling in a way that conforms to both the
best practices from the Gnocchi and Telemetry communities as well as with
those of the OpenStack-Ansible community.

This involves adding support for the Gnocchi services, and the Gnocchi
client[2] into the appropriate hosts and containers. It also involves the
optional configuration of Ceilometer to use Gnocchi in lieu of the currently
supported MongoDB storage solution.

The proposed changes include:

* Creation of an openstack-ansible-os_gnocchi repository and Ansible role
  to support the deployment and management of Gnocchi services.
* Tests to verify the new Ansible role and the integration with OpenStack
  services.
* Documentation to support the role's operation and common deploy
  configurations.

Alternatives
------------

None


Playbook/Role impact
--------------------

Test playbooks will be placed in the openstack-ansible-os_gnocchi repository
for functional testing purposes. Some changes are anticipated within the
openstack-ansible-os_ceilometer role to configure deployment options needed
for an optional integration where Ceilometer uses Gnocchi as it's storage
engine. Further a playbook, necessary group_vars and env and conf profiles
would be provided for the openstack-ansible repository to complete the
integration.


Upgrade impact
--------------

None. While an operator who had previously deployed Ceilometer might be
interested in deploying Gnocchi, there is no migration model for exporting
data from Ceilometer internal storage to Gnocchi storage. The Gnocchi role
and any supporting changes to the Ceilometer role would make the transition
as simple as running the associated playbooks.


Security impact
---------------

None.


Performance impact
------------------

The underlying storage used for Gnocchi would experience higher traffic, which
might require a deployer to account for that additional traffic through
additional tuning.

No other performance impacts are expected.


End user impact
---------------

OpenStack users would be able to make use of Gnocchi as a multi-tenant high
volume timeseries data store when deployers use the role and associated
playbook to deploy Gnocchi.


Deployer impact
---------------

This work provides an optional role for use in the OpenStack-Ansible tooling
for use in their environments.


Developer impact
----------------

Some conditionals may be introduced into the Ceilometer role to facilitate the
clean deployment of Gnocchi as a storage engine for Ceilometer. All other
changes would be self-contained or limited to the introduction of new
variables and a playbook which should present little to no additional
cognitive load for developers.


Dependencies
------------

Only a MySQL-compatible RDBMS is proposed for the Gnocchi index. Only
filesystem, Ceph, and Swift storage engines are proposed. All of these are
currently available within OpenStack-Ansible.

No support is proposed at this time for use of Graffana as a dashboard or the
use of the statsd service endpoints for Gnocchi. In this way we avoid
introducing any new dependencies.


Implementation
==============

Assignee(s)
-----------

Primary assignee:
  Steve Lewis (IRC: stevelle)

Other contributors:
  None


Work items
----------

#. Ask for the new repository, openstack-ansible-os_gnocchi, to be created
#. Create the role for Gnocchi support:

  * Add support for running Gnocchi services (api and metricd) with basic
    convergence testing.
  * Add an Ansible module to leverage gnocchiclient, which is the command line
    interface tool for using and managing Gnocchi.
  * Introduce a playbook for deploying Gnocchi with OpenStack-Ansible.
  * Add support for Gnocchi as an optional storage engine for Ceilometer.
  * Add a full scenario test (described below) to ensure successful
    integration of Gnocchi.
  * Add documentation to the role, and possibly general documentation to the
    install guide for deploying Gnocchi with each of the various supported
    storage engines.

Testing
=======

In an environment where the role is integration tested through the
openstack-ansible repository in one monolithic stack, the additional effort of
deploying this additional project could add as much as a few minutes to gate
testing. That is not desirable.

To preclude the need for that additional step in the main gate, a longer
scenario test is proposed for inclusion in the openstack-ansible-os_gnocchi
role, to integrate with Keystone, Nova, Cinder, Glance, Neutron, Ceilometer,
and Gnocchi with metrics collection enabled and with Nova being exercised to
ensure metering data propagates through the OpenStack environment.

This can then be verified through the use of the Ansible module for
gnocchiclient by querying for the expected measures.

Documentation impact
====================

Role-specific documentation describing the configuration of Gnocchi will be
required.


References
==========

* [1] Gnocchi: http://gnocchi.xyz/
* [2] Gnocchi client:
  http://git.openstack.org/cgit/openstack/python-gnocchiclient/

