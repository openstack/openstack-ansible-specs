Modularizing Neutron plays for agents and non ml2 plugin support
################################################################
:date: 2015-09-09 18:00
:tags: neutron, plugins, agents

This spec is propsed to enhance the current neutron playbooks that take a
static approach to plugin and agent insertion. Where ml2 and a few agents
are used by default.

* https://blueprints.launchpad.net/openstack-ansible/+spec/modularize-neutron-liberty

Problem Description
====================

Presently a straightforward approach does not exist to add new plugins and add
/ remove agents to the neutron setup. A deployer either has to perform these
changes after the whole setup is complete or make their own changes in the
playbooks locally. This feature has already been implemented in juno branch.

Proposed Change
====================

The files in playbooks/roles/os_neutron/tasks will be modified, particularly
neutron_pre_install.yml and  neutron_post_install.yml. Addition of new
parameters will be made to playbooks/roles/os_neutron/defaults/main.yml

Playbook Impact
---------------

The following playbooks are expected to be modified to support this feature:

- playbooks/roles/os_neutron/defaults/main.yml
- playbooks/roles/os_neutron/tasks/main.yml
- playbooks/roles/os_neutron/tasks/neutron_pre_install.yml
- playbooks/roles/os_neutron/tasks/neutron_post_install.yml

Upgrade Impact
--------------

None

Alternatives
------------

Using the current architecture, prospective new core plugins which are not
ml2 will have to take an overwriting the default configuration, after its
done, approach to insert their own changes.

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
plugins/agents from, allowing a wider use case for the OSAD playbooks.

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
  https://launchpad.net/~javeria-ak  ``javeriak``


Work items
----------

This change will include modifying the existing os_neutron role to pick
up what plugin to install along with what agents. The names and configs for
individual plugins will be created as new variables in
playbooks/roles/os_neutron/defaults/main.yml

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

* https://blueprints.launchpad.net/openstack-ansible/+spec/modularize-neutron-plays

