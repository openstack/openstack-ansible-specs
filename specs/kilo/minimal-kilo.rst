Minimal Kilo
############
:date: 2015-03-17 21:34
:tags: kilo, minimum, update,

Update master to point to the minimum configuration nessisary for
a functional kilo stack.

* https://blueprints.launchpad.net/openstack-ansible/+spec/minimal-kilo

This spec is being created to track the work required to get a minimum
viable deployment of kilo. Because the Kilo release of OpenStack has not
yet been released the work done within this blueprint will pull from the
head of master and stabilize on the a given sha for the time being.


Problem description
===================

Master is setup to deploy Juno at this time we want the master branch
to begin tracking Kilo.


Proposed change
===============

In order to have a minimally functional Kilo stack there are several issues
that need to be resolved which have been raised within Launchpad. Once
the following issues are resolved Kilo should be a functional deployment
from the stand point of gating. The point of this Spec is to introduce the
least amount of changes into the stack in an effort to enable a Kilo code
base. The changes should pass gating from the a commit basis. Once this
spec is complete other work can follow to make Kilo a production ready
product.


Alternatives
------------

There are no alternatives to this approach. Without a bulk commit to address
the minimal changes to get Kilo functional we will not be able to move forward
with development.


Playbook impact
---------------

There will be no impact on the playbooks. These changes are on the dependency
and role level which only impact the configuration files and role options.


Upgrade impact
--------------

This change will impact upgrades. The change will introduce new code which
will allow the system to upgrade inplace. That said, this is a transitional
spec which will translate into future work to make Kilo a production ready
product. Upgrades are out of the scope of this spec and it is expected that
Juno to Kilo upgrades will be broken at this point.


Security impact
---------------

These changes will introduce BETA code which will likely have consequences
regarding security however the changes are not geared at production at this
time and will be revised in a fast follow effort.


Performance impact
------------------

Because the Kilo code base is not tested and released the performance of
the stack will not be in scope at this time. As future work develops to
finalize the roles used in Kilo work will be done on a per role basis to
ensure performance.


End user impact
---------------

N/A


Deployer impact
---------------

As stated previously, this change will introduce new BETA code. Deployers
shouldn't be using master at this time.


Developer impact
----------------

This change is geared at enabling developers to begin working on Kilo.


Dependencies
------------

The spec will introduce a number of new dependencies. At this time not all are
exactly known. However, we can safely say that all new clients will be used
throughout the stack as well as various middlewares.


Implementation
==============

Assignee(s)
-----------

Primary assignee:
  https://launchpad.net/~kevin-carter

Other contributors:
  https://launchpad.net/~nolan-brubaker

IRC: cloudnull, palendae


Work items
----------

In order to have a minimum viable installation of OpenStack Kilo
the following issues will need to be addressed.

* `#1428421`_  Keystone.py needs to be updated for kilo
* `#1428431`_  OpenStack Clients need to be updated for Kilo
* `#1428437`_  Update/Removal of pinned Oslo Messaging and Middleware for kilo
* `#1428445`_  Neutron needs plugin references removed for kilo
* `#1428451`_  Heat policy.json file needs to be updated for Kilo
* `#1428469`_  Neutron rootwarp(s) need to be updated for Kilo
* `#1428639`_  Nova requires python-libguestfs in Kilo

.. _#1428421: https://bugs.launchpad.net/openstack-ansible/+bug/1428421
.. _#1428431: https://bugs.launchpad.net/openstack-ansible/+bug/1428431
.. _#1428437: https://bugs.launchpad.net/openstack-ansible/+bug/1428437
.. _#1428445: https://bugs.launchpad.net/openstack-ansible/+bug/1428445
.. _#1428451: https://bugs.launchpad.net/openstack-ansible/+bug/1428451
.. _#1428469: https://bugs.launchpad.net/openstack-ansible/+bug/1428469
.. _#1428639: https://bugs.launchpad.net/openstack-ansible/+bug/1428639


Testing
=======

No changes to the current testing and or gating framework will be made. The
minimum viable Kilo deployment will be required to pass the same gate tests
as are required by our production systems.


Documentation impact
====================

This change specifically does not have any documentation impact.


References
==========

N/A
