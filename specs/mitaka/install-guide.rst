Installation Guide
##################
:date: 2015-11-02 22:00
:tags: install, config, architecture

https://blueprints.launchpad.net/openstack-ansible/+spec/install-guide

Improve the installation guide to appeal to more potential deployers.


Problem description
===================

The current installation guide mainly supports only one rather complex
deployment architecture that limits the apparent flexibility and appeal
of the project to potential deployers.


Proposed change
===============

Improve the installation guide to offer several useful deployment
architectures ranging from simple to complex.

Alternatives
------------

Continue using the existing content that contains significant technical
debt from decisions made prior to entry into the Stackforge and later
OpenStack namespaces.

Playbook/Role impact
--------------------

None.

Upgrade impact
--------------

None, although a separate specification should address development of
upgrade documentation referencing the deployment architectures in the
installation guide as necessary.

Security impact
---------------

None, although the deployment architectures should implement security
measures as necessary.

Performance impact
------------------

None, although more complex deployment architectures could perform poorly
on hardware that disregards minimum requirements.

End user impact
---------------

None.

Deployer impact
---------------

A variety of different deployment architectures ranging from simple to
complex highlight the flexibility of this project and increase appeal to
potential deployers.

Developer impact
----------------

Developers should understand these deployment architectures and adjust them
as necessary to account for new services, changes to existing services,
changes to infrastructure requirements, etc.

Dependencies
------------

None.


Implementation
==============

Assignee(s)
-----------

Primary assignee:
  None

Other contributors:
  None

Work items
----------

* Develop several deployment architectures that range from simple to
  complex and attempt to minimize opinions regarding OpenStack service
  configuration and operation. For example:

  * A simple architecture may include a minimum of two infrastructure
    nodes and one compute node using three networks with minimal physical
    network redundancy and deploy only core OpenStack services.

  * A complex architecture may include a minimum of three infrastructure
    nodes, one compute node, and three storage nodes using four networks
    with reasonable network redundancy and deploy all OpenStack services.

* Potentially restructure the installation guide to implement these
  deployment architectures in the most useful fashion.


Testing
=======

* Verify operation of each deployment architecture prior to each major
  release.


Documentation impact
====================

* Renovating the installation guide.


References
==========

None.
