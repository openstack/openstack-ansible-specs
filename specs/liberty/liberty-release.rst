Liberty Release
###############
:date: 2015-09-08 08:30
:tags: liberty, update

Update the various playbooks and roles in the master branch with the changes
necessary to implement a fully functional and updated Liberty deployment.

* https://blueprints.launchpad.net/openstack-ansible/+spec/liberty-release

Initial work will be based on the Liberty RC tags in each of the OpenStack
projects since Liberty is not yet officially released.


Problem description
===================

While the master branch has been tracking Liberty code for some time, none of
the configuration files have been updated to match the upstream changes in
order to handle deprecation, different default values, etc.


Proposed change
===============

#. Each template/file carried in-tree will need to be reviewed and revised
   according to the new defaults and other adjustments in each OpenStack
   project.

#. Each service will need to be inspected for changes in how deployments
   and upgrades are handled, and the role tasks adjusted accordingly.

#. Any other changes to each service will need to be inspected and adjustments
   to the roles must be made accordingly.

#. In a final test, fatal_deprecations should be set to True for all the
   services to validate that all deprecated configurations have been removed
   or replaced.

The approach to dealing with differences (eg changed defaults for a particular
setting) will be to use the Liberty value where possible. Deployers who are
upgrading from Kilo may use the `config_overrides`_ to implement overrides for
any configurations that they wish to keep at the previous values.

Examples of configs impacted (these will differ depending on the service being
worked on)::

    /etc/<servicename>/<servicename>.conf
    /etc/<servicename>/<servicename>-api-paste.ini
    /etc/<servicename>/policy.json
    /etc/<servicename>/<servicename>-<agentname>.ini

.. _config_overrides: https://docs.openstack.org/project-deploy-guide/openstack-ansible/latest/configure.html

Alternatives
------------

We could, wherever needed, preserve Kilo settings rather than taking forward
the Liberty settings.  This is potentially easier on users in an upgrade
scenario, but does mean that new users deploying Liberty would get an already
out of date deployment. It also means that we miss an opportunity to implement
best practices deployments, instead sticking on old, less relevant, values.


Playbook impact
---------------

There will be no impact on the playbooks. These changes are on the dependency
and role level which only impact the configuration files and role options.


Upgrade impact
--------------

This change will impact upgrades, but upgrades are specifically out of scope
and will be addressed separately in
https://blueprints.launchpad.net/openstack-ansible/+spec/liberty-upgrade-path

The focus for this spec will be for new deployments only.


Security impact
---------------

Security testing and improvements are specifically out of scope. Testing for
security changes and improvements can be done after the release and
implemented in subsequent patches.


Performance impact
------------------

Performance testing and improvements are specifically out of scope. Testing
for performance changes and improvements can be done after the release and
implemented in subsequent patches.


End user impact
---------------

N/A


Deployer impact
---------------

Impacts must be noted in the commit messages for each change.


Developer impact
----------------

This change is to allow development of a production grade Liberty deployment


Dependencies
------------

#. There are several feature blueprints which are expected to merge into the
   master branch alongside these changes. These features are to facilitate
   future improvements in the Mitaka development timeframe and may be
   backported to Liberty. The feature blueprints are marked as dependent in
   Launchpad.

#. The final release of OpenStack-Ansible's Liberty release is entirely
   dependent on OpenStack's Liberty release.


Implementation
==============

Assignee(s)
-----------

* Ceilometer: https://launchpad.net/~miguel-cantu  ``alextricity``

* Cinder: https://launchpad.net/~jesse-pretorius  ``odyssey4me``

* Glance: https://launchpad.net/~jesse-pretorius  ``odyssey4me``

* Heat: https://launchpad.net/~jesse-pretorius  ``odyssey4me``

* Horizon: https://launchpad.net/~steve-lewis  ``stevelle``

* Keystone: https://launchpad.net/~jesse-pretorius  ``odyssey4me``

* Neutron: https://launchpad.net/~jesse-pretorius  ``odyssey4me``

* Nova: https://launchpad.net/~jesse-pretorius  ``odyssey4me``

* Swift: https://launchpad.net/~jesse-pretorius  ``odyssey4me``

* Tempest: https://launchpad.net/~jesse-pretorius  ``odyssey4me``


Work items
----------

See Assignees.

Testing
=======

No changes to the current testing and or gating framework will be made. Each
change that is made to a service to bring forward new configs and settings will
be required to pass the same gate tests as are required by our production
systems.


Documentation impact
====================

All changes made will require DocImpact tags in the commit messages in order
to track the changes required for documentation.


References
==========

* https://etherpad.openstack.org/p/liberty-config-changes

