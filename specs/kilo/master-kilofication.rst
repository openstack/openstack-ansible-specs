Master Kilofication
###################
:date: 2015-03-23 13:00
:tags: kilo, update,

Update the various openstack-ansible playbooks and roles in the master
branch with the changes necessary to implement a fully functional and updated
kilo deployment.

* https://blueprints.launchpad.net/openstack-ansible/+spec/master-kilofication

Initial work will be based on the k3 tags in each of the openstack projects
since kilo is not yet officially released.

Problem description
===================

Master is setup to deploy Juno at this time we want the master branch to begin
tracking Kilo.


Proposed change
===============

As opposed to the minimal-kilo blueprint which is focused on making the minumum
fewest possible changes necessary to point at kilo and have a deployment that
passes gating, this specification is targeted more at updating all config files
and code to bring in the kilo versions of the configs for each service, parsing
each file for differences and making informed decisions about what values to
take to ensure we have a production grade deployment system.

The approach to dealing with differences (eg changed defaults for a particular
setting) will be to use the kilo value where possible, adding an option to
make any changed setting tunable if it was not already. This gives the option
to users who are upgrading from juno to be able to reset a value back to the
juno default if desired, but also means that greenfield deployments of kilo use
the (hopefully better) kilo value.

Examples of configs impacted (these will differ depending on the service being
worked on)::

    /etc/<servicename>/<servicename>.conf
    /etc/<servicename>/<servicename>-api-paste.ini
    /etc/<servicename>/policy.json
    /etc/<servicename>/<servicename>-<agentname>.ini



Alternatives
------------

We could, wherever needed, preserve juno settings rather than taking forward
the kilo settings.  This is potentially easier on users in an upgrade scenario,
but does mean that new users deploying kilo would get an already out of date
deployment. It also means that we miss an opportunity to implement best
practices deployments, instead sticking on old, less relevant, values.


Playbook impact
---------------

There will be no impact on the playbooks. These changes are on the dependency
and role level which only impact the configuration files and role options.


Upgrade impact
--------------

This change will impact upgrades, but upgrades are out of scope for this spec
which will be addressed separately.  Largely it addresses greenfield
deployments of kilo.


Security impact
---------------

These changes will initially be based on BETA code (k3 and rc1 tags of kilo)
which may have consequences regarding security, but work will be done to test
against production kilo when it is released (and prior to the 11.0.0 release
of openstack-ansible being tagged)


Performance impact
------------------

Because the Kilo code base is not tested and released, the performance of the
stack will not be in scope at this time. As future work develops to finalize
the roles used in Kilo, work will be done on a per role basis to ensure
performance.


End user impact
---------------

N/A


Deployer impact
---------------

As stated previously, this change will initially introduce new BETA code.
Deployers shouldn't be using master at this time.


Developer impact
----------------

This change is to allow development of a production grade kilo deployment


Dependencies
------------

The spec will introduce a number of new dependencies. At this time not all are
exactly known. However, we can safely say that all new clients will be used
throughout the stack as well as various middlewares.


Implementation
==============

Assignee(s)
-----------

Various

Work items
----------

Unknown at this time

Testing
=======

No changes to the current testing and or gating framework will be made. Each
change that is made to a service to bring forward new configs and settings will
be required to pass the same gate tests as are required by our production
systems.


Documentation impact
====================

This change will likely have documentation impact. Specifically when
documenting changed values or deprecated config items.


References
==========

N/A
