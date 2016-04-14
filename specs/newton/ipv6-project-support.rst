IPv6 Project Support
####################
:date: 2015-09-09 22:00
:tags: ipv6

ospenstack-ansible should support IPv6 for project networks.
To that effect we should make sure that the necessary components and
configurations are installed so that openstack can expose and route IPv6 for
project networks.


Problem description
===================

Neutron currently (in kilo) has the ability to to manage and route IPv6 data.
OpenStack Ansible currently has a few holes in IPv6 support on Neutron tenant
networks (not installing the radvd package in the neutron-agents container
for instance).


Proposed change
===============

Add a test case for proving IPv6 access on project networks works as expected


Alternatives
------------

Don't explicitly support IPv6


Playbook impact
---------------

As the primary change is adding a test case this is somewhat open ended.
As the support for IPv6 via Neutron is already mostly there this should be
low impact, will likely only be adding the missing package and test support.


Upgrade impact
--------------

None


Security impact
---------------

Low, at the moment the only known change is to ensure that radvd is installed
so that Neutron can configure/control it.


Performance impact
------------------

None


End user impact
---------------

The end user will be able to configure IPv6 in the project networks.


Deployer impact
---------------

None


Developer impact
----------------

None once spec is implemented.


Dependencies
------------

None


Implementation
==============

Assignee(s)
-----------

Primary assignee:
  prometheanfire


Work items
----------

* add test support for IPv6 in OpenStack Ansible

  * This would be via configuring a RFC4193 network and connecting from the
    neutron radvd namespace to the instance.

  * It would also test unicast routing between neutron networks using RFC4193.

* ensure that tests pass


Testing
=======

Ensure that the instance gets an IP in a certian address space and can ping
the gateway.

Test for routability, ping between instances on two neutron network segments.


Documentation impact
====================

Should be minimal


References
==========

https://bugs.launchpad.net/openstack-ansible/+bug/1492080
