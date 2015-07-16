Add support for SystemD
#######################
:date: 2015-07-14
:tags: systemd

The purpose of this spec is to adjust our current upstart only init process to allow
us to leverage SystemD. While SystemD is not present within the Ubuntu 14.04 LTS
OS that we use today it is something that is coming within the next LTS release and
something that we should begin implementing as an alternative to upstart.

https://blueprints.launchpad.net/openstack-ansible/+spec/add-support-for-systemd


Problem description
===================

OSAD presently only support Ubuntu 14.04 LTS using upstart. In the next LTS upstart
will no longer be an option. For this reason I believe its time to begin implementing
SystemD support within the OpenStack roles.


Proposed change
===============

The basic change is more of a structural one. Essentially adding SystemD support will
be a new template and will follow much of the same pattern found within our current
upstart process.


Alternatives
------------

n/a - SystemD is coming and the sooner we have an oppinion on it the better off we will
be.


Playbook impact
---------------

The playbooks will not be impacted however the roles will have a new SystemD template and
set of tasks that will enable the ability for the system to use SystemD.


Upgrade impact
--------------

Adding in SystemD support will ensure that deployers are able to upgrade to future OS's
that only have SystemD available.


Security impact
---------------

n/a


Performance impact
------------------

n/a


End user impact
---------------

n/a


Deployer impact
---------------

n/a


Developer impact
----------------

n/a


Dependencies
------------

n/a

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  https://launchpad.net/~kevin-carter ``cloudnull``


Work items
----------

* Add SystemD templates to all OpenStack roles.
* Add SystemD tasks to all OpenStack roles.


Testing
=======

Being that we do not gate on anything that uses SystemD at the moment this
will be a set of changes that are being implemented to future proof OSAD.
This change will also allow us to being looking into "other" OS support
which will likely carry with it an implementation of SystemD, such as Debian
"Jessie".


Documentation impact
====================

n/a


References
==========

n/a
