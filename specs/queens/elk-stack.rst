ELK Stack
#########
:date: 2017-12-11 11:00
:tags: logging, monitoring, operations

Blueprint on Launchpad:

  * https://blueprints.launchpad.net/openstack-ansible/+spec/elk-stack

Log file analysis is an important part of maintaining and troubleshooting
OpenStack clouds, but using traditional single server methodology to analyze
the logs on clouds with tens, hundreds or thousands of servers can become
problematic and unwieldy.  By leveraging the search, collation and analysis
features of the ELK (Elasticsearch`[1]`_, Logstash`[2]`_ and Kibana`[3]`_) stack
we can provide a cloud level view of all of the log files. The ELK stack also
provides the ability to correlate log messages across various services, perform
detailed log analysis and do trending based on metrics derived from log
messages.

Problem description
===================

For deployers and operators findings specific events in the myriad log files
produced by the various OpenStack, system and ancillary services can be tedious
and error prone.  With traditional tools the possibility of missing critical log
entries grows as the size of the cluster increases. Log file analysis provides
vital information about the state of the OpenStack services as well as the
underlying hardware.  Currently there are no tools provided by OpenStack-Ansible
to detailed log analysis, correlation and trending.

Proposed change
===============

Utilizing the logging/utility node we install the ELK stack in containers, logs
are shipped from the individual nodes/containers using the Filebeat package.
Using Filebeat to perform the initial log shipping allows us to do initial
multiline parsing distributing the load away from a single Logstash container.
Version requirements of the ELK packages will be maintained in the ELK roles and
barring security fixes the major version of those packages should not change
during the release cycle of Openstack.  The ELK roles are consumed via Ansible
Galaxy pointing to specific SHAs.

Notable changes:
  * Create 3 containers on the logging/utility node, one each for Elasticsearch,
    Logstash and Kibana. (Additional containers can be created to facilitate HA if
    needed.)
  * Install the Filebeat package on all nodes/containers
  * ELK and Filebeats galaxy role SHAs added to `ansible-requirements.yml`


Alternatives
------------

Logs are currently shipped to a centralized rsyslog-server container on the
logging/utility server allowing for some sort of centralized log parsing using
command line utilities.  There are other 3rd party solutions with various levels
of cost, adoption and support.

Playbook/Role impact
--------------------

The changes required are located in stand alone playbooks.  Additional roles
will need to be created for Logstash, Kibana and Filebeat, the
`ansible-elasticsearch``[4]`_ maintained by elastic.co provides Elasticsearch.
Configuration can be stand-alone or integrated into the `user-variables.yml` and
`user-secrets.yml` files.


Upgrade impact
--------------

As this is the initial implementation there is no upgrade impact.  Future
versions will require upgrade planning as it may be necessary to upgrade
versions of the ELK packages, OpenJDK packages and possibly the Elasticsearch
database itself.


Security impact
---------------

This software provides a web based front end as well as API access to any
information contained in the Openstack, service and system logs that are
shipped to it. As such it will need to be only visible to authenticated users.
All access can be secured through the traditional hardening that is applied to
any standard web service, namely TLS and an authentication mechanism.
Furthermore since the ELK stack is behind a VIP we can limit access to certain
IPs and/or networks via a number of ACLs.

By default logs are shipped in plaintext, it is possible, however, to enable
SSL encryption on this transport should it be needed.


Performance impact
------------------

Based on testing and real-world analysis the largest performance impact will be
on the logging/utility server. As this devices original intent was to perform
log processing this is expected and not unusual.  The filebeat service running
in each node/container has demonstrated a negligible performance impact, but
certain best practices such as limiting logging levels and eliminating
tracebacks in the logs will help maintain the light footprint.  Filebeat should
not impact the operation of any Openstack services as it is simply a log file
processor/shipper, although network utilization could be a concern should debug
logging be enabled on a particularly busy service.

Elastic.co is the maintainer of all of the software other than Java, which is
maintained by Oracle corporation.  Both of these entities provide enterprise
software and thus follow strict release schedules and have reliable upstream
repositories for their software.


End user impact
---------------

End users should not notice the changes from this work. This is primarily
intended for deployers and operators.  This change does give operations teams
more insight into the environment and will hopefully facilitate a more
performant and stable deployment.


Deployer impact
---------------

The ELK stack is an optional component and does not directly interact with any
Openstack services.  All of the ELK packages are provided via apt/yum
repositories.  An additional secret will need to be created for the `kibana`
user.  The filebeat package will be installed in all containers and on all
nodes but it is extremely lightweight, with configuration stored in
`/etc/filebeat`.  Java is required for ELK so the `openjdk` (default) or JDK
implementation of the deployers choosing will need to be installed in three
containers on the logging/utility node.


Developer impact
----------------

This should be a minimal change for developers, the one thing that they will
need to keep in mind is if additional log files are added they will need to be
added to the filebeat configuration, this can be handled by re-running the
filebeat play against the containers with the new logs.


Dependencies
------------

There are no dependencies.

Implementation
==============

Assignee(s)
-----------

Primary Assignee:
  David Wilde (d34dh0r53)


Work items
----------

1. Create ELK and filebeats roles in openstack-ansible, these roles will be
   generic enough to be published to ansible-galaxy so that they are usable
   by the Ansible community at large.
2. Create playbook(s) to install the ELK stack and filebeats, these playbooks
   will install the OpenStack specific configuration and parsing files.
3. Create testing procedures for the stack
4. Documentation

Testing
=======

The ELK stack should be tested on each commit by ensuring that the services
start and that logs are flowing into the system and being parsed correctly.
This can be acomplished by injecting a line into a services log file and then
using the elasticsearch API via curl to verify that the line was correctly
inserted into the database with the expected fields parsed.


Documentation impact
====================

Along with the general installation procedures and configuration the key points
of documentation will be:
  * Filebeats parsing rules
  * Logstash parsing rules
  * Kibana dashboard configuration
  * The default Kibana dashboard
  * Performance impact and tuning of the ELK stack


References
==========

[1] https://elastic.co/products/elasticearch

[2] https://elastic.co/products/logstash

[3] https://elastic.co/products/kibana

[4] https://github.com/elastic/ansible-elasticsearch
