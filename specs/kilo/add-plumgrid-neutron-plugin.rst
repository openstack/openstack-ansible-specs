Add PLUMgrid plugin to neutron playbooks
########################################
:date: 2015-06-2 14:30
:tags: neutron, plugins, networking

This spec is propsed to insert the capability of using the PLUMgrid
OpenStack Neutron Plugin through the os-ansible neutron playbooks.

* https://blueprints.launchpad.net/openstack-ansible/+spec/add-plumgrid-neutron-plugin

Problem Description
===================

PLUMgrid is a core neutron networking plugin that has been a part of OpenStack
neutron since Grizzly. It offers a Network Virtualization Platform that uses
direct communication with the Hypervisor layer to provide all the networking
functionality requested through Neutron APIs. The PLUMgrid Neutron Plugin
implements Neutron v2 APIs and helps configure L2/L3 virtual networks
created through the PLUMgrid Platform. It also implements External Networks
and Port Binding Extensions.

APIs supported by the PLUMgrid plugin:
 - Networks
 - Subnets
 - Ports
 - External Networks
 - Routers
 - Security Groups
 - Quotas
 - Port Binding
 - Provider Networks

Proposed Change
===============

This change is proposed to add the PLUMgrid plugin as a core plugin option
alongside ml2, which will be the default. This configurability should already
be achieved by the BP: modularize-neutron-plays.

The rest of the installation for PLUMgrid that requires PLUMgrid Controller
and Compute components, that enable management of the plugin, will be added
externally through Ansible Galaxy roles.

The changes described below assume the previously mentioned BP modularization
changes in place.

This feature is proposed for both kilo and juno branches, the juno change
will be carried out first:

1. For juno, parameters relevant to the PLUMgrid plugin, namely the plumgrid
   core plugin and plugin config file, plumgrid.ini will be added to a new
   dictionary item in 'neutron_plugins' in
   inventory/group_vars/neutron_all.yml This will allow setting the
   'neutron_plugin_type = plumgrid' if desired.

2. For kilo, parameters relevant to the PLUMgrid plugin will be added to a
   new dictionary item in 'neutron_plugins' in
   'playbooks/roles/os_neutron/defaults/main.yml'. This will allow setting the
   'neutron_plugin_type' to plumgrid if desired.

Playbook Impact
---------------

1. In juno, the following files are expected to be modified:

 - rpc_deployment/playbooks/openstack/inventory/group_vars/neutron_all.yml

The following templates will be created in neutron-common role:

 - rpc_deployment/roles/neutron_common/templates/plugins/plumgrid/plumgrid.ini
 - rpc_deployment/roles/neutron_common/templates/plugins/plumgrid/plumlib.ini
 - rpc_deployment/roles/neutron_common/templates/rootwrap.d/plumlib.filters

2. In kilo, these files are expected to be modified:

 - playbooks/roles/os_neutron/defaults/main.yml

New templates will be added in the os_neutron role:

 - playbooks/roles/os_neutron/templates/plugins/plumgrid/plumgrid.ini
 - playbooks/roles/os_neutron/templates/plugins/plumgrid/plumlib.ini
 - playbooks/roles/os_neutron/files/rootwrap.d/plumlib.filters

Upgrade impact
--------------

None

Alternatives
------------

To continue using the default ml2 and linuxbridge-agent neutron deployment
with no possibility of other core neutron plugins.

Security Impact
---------------

N/A

Performance Impact
------------------

This change is not expected to impact performance. A typical PLUMgrid plugin
installation, will furthermore disable neutron agent installations. Hence the
overall performance is expected to remain the same.

End User Impact
---------------

End users will be able to leverage the enhanced scale and operational capabilities
provided by the PLUMgrid plugin when choosing to install this plugin. Further details
can be found in the References section below.

Deployer Impact
---------------

This will provide Deployers with the option to use PLUMgrid as the neutron
plugin. Upgrading from a previous release to use this new feature will only
be possible through a re-run of the neutron playbooks as well. This change
does not effect running instances within the cloud.

Developer Impact
----------------

This change adds further installable options and as such does not
effect the default flow of the playbooks.


Dependencies
------------

None

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  https://launchpad.net/~javeria-ak

Work items
----------

This change will use the modularized neutron playbooks to provide
PLUMgrid as a plugin option. A set of three new template files will
be added to the neutron plays to support plumgrid.

Dependencies
------------

Dependent on:

- https://review.openstack.org/184665
- https://blueprints.launchpad.net/openstack-ansible/+spec/modularize-neutron-plays

Testing
=======

There are no additional changes required to test this in the current testing
and or gating framework that also covers the neutron components.

Documentation Impact
====================

Documentation describing how to modify the configuration parameters
to install PLUMgrid will be required. This will be deployer documentation.

References
==========

* http://www.plumgrid.com/

* https://wiki.openstack.org/wiki/PLUMgrid-Neutron
