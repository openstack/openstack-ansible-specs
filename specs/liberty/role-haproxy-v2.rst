HAProxy improvements
#################################
:date: 2015-09-04 14:00
:tags: haproxy, production use

HA Proxy can be improved by adding a few changes:

* Making it really HA
* Allowing configuration interface to easily adapt load
* Deploying only the configuration for the services
  deployed within the inventory.
* Improving backends configuration, for example galera or
  adapting the timer values to be more efficient

https://blueprints.launchpad.net/openstack-ansible/+spec/role-haproxy-v2

Problem description
===================

There are a few features already asked by the community:

* HA for haproxy
* Enable statistics and improve manageability of haproxy
* Limiting the unnecessary checks of haproxy


Proposed change
===============

* Implement keepalived for haproxy
* Change the standard haproxy role to add
  administrative tools (admin level on socket and stats)
* Remove the large haproxy variable in vars/ folder
* Give this information component by component
  (in the group_vars), and make it possible to have
  user overrides (user_variables or component by component).
  Then ``delegate`` the configuration to haproxy hosts.
* Introduce a skip variable, if you want to deploy
  haproxy on some components but not some others

Alternatives
------------

Wait for ansible2 to have variable merging/cleanup for dicts
on a per task/playbook basis.

Playbook/Role impact
--------------------

The playbook ``haproxy-install.yml`` will be completely
overwritten.

haproxy playbook run will be longer, due to the ``delegate to``.


Upgrade impact
--------------

None.

Security impact
---------------

No change

Performance impact
------------------

Improved performance by:

* Doing less unnecessary checks to backends
* Adding an easy way to set customer values for the
  backend's timers.

End user impact
---------------

No change

Deployer impact
---------------

* No change in default configuration
* The deployer can overwrite the
  ``haproxy_service_configs`` per component

Developer impact
----------------

No impact at first sight.

Dependencies
------------

None

Implementation
==============

Assignee(s)
-----------

None

Work items
----------

* Keepalived:
  https://review.openstack.org/#/c/217517/
* Easy administration:
  https://review.openstack.org/#/c/215019/ and https://review.openstack.org/#/c/214110/
* Default configuration less static:

  * rewrite haproxy-install with the "delegate_to" and
    with a "when" haproxy_component_skip (if you want to deploy
    haproxy on some components but not some others)
  * create a file per component with default variables under group_vars

* Default timer value changes.

Testing
=======

* Does this change impact how gating is done?

There will be a change to haproxy-install playbook if merged.

* Can this change be tested on a **per-commit** basis?

Yes

* Given the instance size restrictions, as found in OpenStack Infra
  (8GB Ram, vCPUs <= 8), can the test be run in a resource constrained
  environment?

No change

* Is this untestable given current limitations (specific hardware /
  software configurations available)? If so, are there mitigation plans
  for this change to be tested within 3rd party testing, gate enhancements,
  etc...?

No

* If the service is not OpenStack specific how can we test the change?

Running the new playbooks


Documentation impact
====================

For those who change the default configuration of haproxy (currently not
documented), this change would modify their current configuration, so
it needs to be documented. Explanation of the skip variable and component
by component override should be good to add in the doc too.

References
==========

None
