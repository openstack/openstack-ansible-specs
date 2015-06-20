Modularizing Neutron plays for agents and non ml2 plugin support
################################################################
:date: 2015-03-30 16:35
:tags: neutron, plugins, agents

This spec is propsed to enhance the current neutron playbooks that take a
static approach to plugin and agent insertion. Where ml2 and a few agents
are used by default.

* https://blueprints.launchpad.net/openstack-ansible/+spec/modularize-neutron-plays

Problem Description
====================

Presently a straightforward approach does not exist to add new plugins and add
/ remove agents to the neutron setup. A deployer either has to perform these
changes after the whole setup is complete or make his own changes in the
playbooks.

Proposed Change
====================

This feature is proposed for both master and juno branches, the juno
effort will be carried out first:

1. For juno, the openstack/roles/neutron_common.yml will be modified to
install a configurable list of plugins and agents through new variables
defined in inventory/group_vars/neutron_all. The default values to these
new variables with be the current set of installed agents and plugins.

2. For master, the playbooks/roles/os_neutron/tasks files will be modified,
particularly neutron_post_install.yml. Addition of new parameters will be made
to playbooks/roles/os_neutron/defaults/main.yml

Playbook Impact
---------------

1. In juno, the following files are expected to be modified:

 - openstack/roles/neutron_common.yml
 - openstack/inventory/group_vars/neutron_all.yml

2. In master, these files will be modified:

 - playbooks/roles/os_neutron/tasks/neutron_post_install.yml
 - playbooks/roles/os_neutron/defaults/main.yml

Upgrade Impact
--------------

None

Alternatives
------------

Using the current architecture, prospective new plugins which are not ml2 will
have to take an overwriting the default configuration, after its done,
approach to insert their own changes.

Security Impact
---------------

None known at this time.

Performance Impact
------------------

This change is not expected to impact performance. Installing the default set
of agents and plugins as done now, will take the same amount of effort.

End User Impact
---------------

This is not expected to impact end users as it deals with the deployment aspect
only.

Deployer Impact
---------------

This will introduce a more modular architecture for deployers to select neutron
plugins/agents from, allowing a wider use case for these playbooks.

Developer Impact
----------------

Using the default values will require no new developer effort, only those
interested in changing the neutron config will be effected.

Dependencies
------------

N/A

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  https://launchpad.net/~javeria-ak


Work items
----------

This change will include modifying the existing neutron_common role to pick
up what plugin to install along with what agents. The names and configs for
individual plugins will be created as new variables in
inventory/group_vars/neutron_all.yml

Dependencies
------------

N/A

Testing
=======

There are no additional changes required to test this in the current testing
and or gating framework.


Documentation Impact
====================

A bit of additional documentation describing how to insert new plugins/agents
will be required. This will be deployer documentation.

References
==========

N/A

