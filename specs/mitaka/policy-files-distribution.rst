Policy Files Distribution to Horizon
####################################
:date: 2015-11-24 15:00
:tags: cross-project, cross-role, json, policy, distribution, Horizon

OpenStack Horizon can use policy.json files to filter the actions
available on its webinterface. For that, Horizon consumes the policy.json
files of each openstack project (like cinder/nova/glance/...),
it doesn't distribute its own.

Therefore, if the deployer wants to have a consistent policy through
the apis and the webinterface, the deployer has to upload its policies
to Horizon.

Currently, it's not done within openstack-ansible.

https://blueprints.launchpad.net/openstack-ansible/+spec/policy-files-distribution

Problem description
===================

Currently every deployer that needs policy files is doing the same work.
Let's try to avoid that in the future: They create policy files
for the openstack project thanks to openstack-ansible but then need
to upload the policy files to Horizon manually with their own role.

This should fix that, and propose a solution to the policy files deployment

Proposed change
===============

First, there should be a generic cross-role switch
(``policy_file_distribution_enabled``) defaulted to False, unless the
deployer has set a ``_policy_overrides`` for a component.
Of course, a deployer can prevent this policy file distribution
by setting it to False.

Then, we should handle the policy distribution in two steps:

1. Download each deployed policy.json file (from the first host of each group)
   during the ``horizon-install`` playbook into the
   ``policy_files_distribution_folder`` (by default ``/etc/openstack_deploy/``)
   on the deploy node.

2. Having the Horizon role could consume these files on the deploy host
   and upload the json files to the Horizon nodes. This would require
   connecting on multiple hosts and will lengthen deployment's
   time (on the first run, if enabled)

Alternatives
------------

* Not implementing this, and let the deployer do the work himself
* Rely on Horizon distributing its own policy mapping in the future
* Include each project's (i.e. nova,neutron,etc.) default policy file from
  their git source in the Horizon role and use the `config_template` to
  upload/override the final ``nova_policy.json, glance_policy.json,...``
  files on Horizon. This would require us to track OpenStack project
  policy changes in both Horizon and the respective project roles.
* Download each project policy.json file from their git source
  repository (i.e. glance, nova,etc.) to the deployment node before
  running the os_horizon role. Then use the `config_template` to
  upload/override the final json files on Horizon. This would require us
  to track OpenStack projects' policy files URL changes.
* Last alternative would be to distribute using another mechanism
  (like memcache/swift/file sync...).

Playbook/Role impact
--------------------

Small changes in playbooks/role.

Upgrade impact
--------------

No upgrade impact.

Security impact
---------------

None

Performance impact
------------------

Slightly longer deployment time if enabled for the first time.
Implementation would redownload if a file exists, unless explicitly
told by a variable: ``policy_file_distribution_force_refresh``.

End user impact
---------------

The end-user will not have inconsistent behaviour of having one button
that doesn't work because the policy prevents it in the component api
but not in Horizon.


Deployer impact
---------------

A few new variables:

* policy_file_distribution_enabled
* policy_file_distribution_force_refresh
* policy_files_distribution_files

NB: Their name could be adapted later
(cf. implementation)

Developer impact
----------------

Nothing should change.

Dependencies
------------

None.

Implementation
==============

Assignee(s)
-----------

None yet

Work items
----------

1. group_var to define auto download
2. playbook edition to download policies
3. role changes to upload json files


Testing
=======

* Does this change impact how gating is done?

No

* Can this change be tested on a **per-commit** basis?

Yes

* Given the instance size restrictions, as found in OpenStack Infra
  (8GB Ram, vCPUs <= 8), can the test be run in a resource constrained
  environment?

Yes.

* Is this untestable given current limitations (specific hardware /
  software configurations available)? If so, are there mitigation plans
  for this change to be tested within 3rd party testing, gate enhancements,
  etc...?

This change is testable.

* If the service is not OpenStack specific how can we test the change?

It's openstack specific


Documentation impact
====================

We'll need to update the documentation to mention how to edit the policies
and how to enable the policy distribution to Horizon.

References
==========

Policy files url:

* Horizon: http://docs.openstack.org/developer/horizon/topics/policy.html#policy-files

* keystone: https://github.com/openstack/keystone/blob/master/etc/policy.json

* Glance: https://github.com/openstack/glance/blob/master/etc/policy.json

* Nova: https://github.com/openstack/nova/blob/master/etc/nova/policy.json

* Neutron: https://github.com/openstack/neutron/blob/master/etc/policy.json

* Cinder: https://github.com/openstack/cinder/blob/master/etc/cinder/policy.json
