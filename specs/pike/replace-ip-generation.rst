Replace IP Generation Code
##########################
:date: 2017-1-11 22:00
:tags: inventory, ip, networking


The current inventory code uses a simple set to manage assigned IPs
(``USED_IPS``) and complex queues to pull from the available subnets.

This code can be simplified and made more modular.

Launchpad blueprint:
  * https://blueprints.launchpad.net/openstack-ansible/+spec/replace-ip-generation

The current IP generation code is tightly couple to the configuration loading,
writing, and inventory manipulation code. To help provide better, more focused
test coverage, this code can be updated and replaced.



Problem description
===================

The current IP generation code is difficult to maintain, despite mostly being
`moved <https://review.openstack.org/#/c/392277/>`_ into a separate ``ip.py``
module. The code uses the external ``Queue`` class, which is slightly more
complex than necessary. The ``USED_IPS`` set and the pools of available IPs
are not managed together, and could easily become out-of-sync.

New code has been written to add an `IPManager class`_, but it is not
currently integrated into any other code. Such integration is a somewhat
large task, and would be error-prone to do in a single review. This spec
is intended to serve as a road map to guide small, focused changes towards
using it.

Note that while the IPManager includes an API for external IPAM systems, this
spec is only focused on using this class within the code, not on any sort of
plugin system.

Proposed change
===============

An initial draft of new IP management code has been written in the `IPManager
class`_.

After that, the existing ``get_ip_address``, and ``set_used_ips`` were
refactored to still use the existing data structures, but in a way that
would allow usage of the new IPManager class. See `review 403915`_.

Some refactors may be necessary for the IPManager class to facilitate this
and further codify assumptions.

Alternatives
------------

The code be left as is, with the assumption that it will be replaced wholesale
by some other system in the near future. That replacement might happen via
plugins or a new inventory codebase. This has not been deeply explored in the
context of the IP management/generation.

One such replacement system, for example, could be using LXD to entirely
manage container creation, which is where IP generation is primarily used.

Playbook/Role impact
--------------------

No noticeable impact on the playbooks and roles should be seen; this is
largely facilitating code maintenance and should produce the same output.

Upgrade impact
--------------

There should be no upgrade impact - the IPManager class should be loaded with
the already-generated IP addresses in upgraded installations.

Security impact
---------------

This change should not affect any sensitive data. It is unrelated to secret
storage.


Performance impact
------------------

Generating IPs may be slightly faster, since this approach doesn't rely on
delayed access from ``Queue`` objects. However, the overall runtime of the
inventory is negligible in the overall speed of the system and hasn't been
profiled.

End user impact
---------------

This change would be invisible to users of the deployed cloud.

Deployer impact
---------------

No configuration or output changes should be introduced. The current
configurations should be used as-is.

Developer impact
----------------

This should improve quality of life for developers debugging the IP generation
behavior.

Dependencies
------------

This has no direct dependencies on other blueprints or specs.

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  nolan-brubaker, IRC: palendae

Other contributors:
    steve-lewis, IRC: stevelle

Please add **IRC nicknames** where applicable.

Work items
----------

* Refactor current IP loading/management functions to be amenable to replacing
  the data structures.

* Replace the data structures and update the objects being passed between
  functions.

Testing
=======

Unit and integration tests should be added for all code changes to confirm
there are no regressions.

Documentation impact
====================

Developer documentation should be updated to reflect the new mechanism used,
preferably included with implementation patches.

References
==========

N/A

.. _`IPManager class`: https://review.openstack.org/#/c/397299/
.. _`review 403915`: https://review.openstack.org/#/c/403915/
