Monasca HA & Monasca-Agent OSA Roles
####################################
:date: 2015-10-07 17:00
:tags: Ansible, Monasca, Monasca-agent, High Availability, Clustering

Currently, the Monasca role for OSA does not configure any of the services in HA. Installation of the monasca-agent into hosts and containers is also not handled.

https://blueprints.launchpad.net/openstack-ansible/+spec/monasca-ha

https://blueprints.launchpad.net/openstack-ansible/+spec/monasca-agent

Problem description
===================

Most modern monitoring systems have the ability to cluster its database for resiliency and present the user interface in a highly available fashion. The Monasca role as currently defined only launches single hosts with the supporting services configured as single hosts. If you wanted to put the monasca-api behind a VIP, you would have three different apis with disparate information.

There is currently no monasca-agent OSA role defined. To get monasca-agent on a OSA installation, you have to manually install it, or use the ansible-monasca-agent playbook to install it into hosts and containers after the fact.

Proposed change
===============

Notable changes:
  * Monasca-api & friends

   * The following services must be placed under a VIP

    * Monasca-api
    * Influxdb
    * MySQL

   * The following services must have clustering configured:

    * MySQL
    * Influxdb
    * Storm Nimbus & Supervisor
    * Storm Nimbus & Supervisor
    * Zookeeper
    * Grafana & Monasca-ui
      * Not clustered, but for HA to work correctly on Grafana, session sharing and configuration database connection must be set to the OSA Galera cluster

   * Add python-monascaclient to the utility containers

  * Monasca-agent

   * The os_monasca-agent role must be developed to install the agent onto all hosts during the pre-install
   * Improvements must be made to monasca-agent to correctly identify all services

Alternatives
------------

Only alternative is using another monitoring system if you plan on using OSA, or living with the eventual doom of your monitoring data.


Playbook/Role impact
--------------------

If the user does not include the os_monasca role, there will be no impact other than the inclusion of the python-monascaclient into the utility containers.
If included, the user will likely have to add a few configuration options, like VIP ranges, replication factors, things that related solely to monitoring and data resiliency.

Upgrade impact
--------------

Monasca & friends work independently from OSA so no impact to the openstack upgrade process is expected.

If the containers are trashed during the upgrade, monasca-agent will have to be re-installed.
If not, monasca-reconfigure script will have to be ran to discover changes to hosts/containers.

Security impact
---------------

* Does this change touch sensitive data such as tokens, keys, or user data?

  * Only sensitive item the playbook would need would be the admin password to enroll the monasca users & endpoints.

* Does this change alter a deployed OpenStack API in a way that may impact
  security, such as a new way to access sensitive information or a new way to
  login?

  * Monasca uses it's own API and only consumes from the keystone API.

* Does this change involve cryptography or hashing?

  * No

* Does this change require the use of sudo or any elevated privileges?

  * Yes but only on the dedicated Monasca hosts to install & configure services. Monasca-agent install does not require escalation.

* Does this change involve using or parsing user-provided data? This could
  be directly at the API level or indirectly such as changes to a cache layer.

  * Only Monasca specific variables will be provided by users.

Performance impact
------------------

Monasca runs on it's own dedicated hosts.

The impact of the monasca-agent on hosts & containers is minimal.

End user impact
---------------

Addition of monitoring as a service. No changes to existing user exposed services.

Deployer impact
---------------

* What config options are being added? Should they be more generic than
  proposed? Are the default values ones which will work well in
  real deployments?

 * Only configuration options related to Monasca will be added.

* Is this a change that takes immediate effect after its merged, or is it
  something that has to be explicitly enabled?

 * Monasca must be explicitly enabled.

* If this change is a new binary, how would it be deployed?

 * N/A

* Please state anything that those doing continuous deployment, or those
  upgrading from the previous release, need to be aware of. Also describe
  any plans to deprecate configuration values or features.  For example, if we
  change the name of a play, how do we handle deployments before the change
  landed?  Do we have a special case in the code? Do we assume that the
  operator will recreate containers within the infrastructure of the cloud?
  Does this effect running instances within the cloud?

 * Does not affect current Openstack Installation without Monasca.
 * Monasca specific upgrade instructions will be provided if needed.


Developer impact
----------------

Will not affect developers not working with Monasca.

Dependencies
------------

No blueprint dependencies.

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  rmelero

Work items
----------

Same as Proposed changes.

Testing
=======

N/A

Documentation impact
====================

Openstack ansible documentation will not be affected.

New os_monasca documentation will be written.


References
==========

https://github.com/b-com/ansible-monasca

