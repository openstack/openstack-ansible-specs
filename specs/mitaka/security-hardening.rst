Security Hardening for OSAD Hosts
#################################
:date: 2015-09-10 00:00
:tags: security

The goal of this spec is to apply hardening standards to openstack-ansible so
that users can build environments that meet the requirements of various
compliance programs, such as the `Payment Card Industry Data Security Standard
(PCI-DSS)`_.  These changes won't make a particular environment PCI compliant,
but they should bring the environment in compliance with Requirement 2.2 from
PCI-DSS.  That requirement states that deployments must follow an
industry-accepted hardening standard.

.. _Payment Card Industry Data Security Standard (PCI-DSS): https://en.wikipedia.org/wiki/Payment_Card_Industry_Data_Security_Standard

Blueprint - Security Hardening for OSAD Hosts:
  * https://blueprints.launchpad.net/openstack-ansible/+spec/security-hardening

Problem description
===================

Compliance programs, such as PCI-DSS, often have a requirement for using
industry-accepted hardening standards for all deployments.  At the moment,
deployments done on Ubuntu 14.04 with openstack-ansible meet many, but not all,
security hardening standards that are approved within PCI-DSS.

PCI-DSS 3.1 Requirement 2.2 states that deployments that handle credit card
data must be secured with industry-accepted hardening standards.  The test of
the requirement is as follows:

    2.2 Develop configuration standards for all system components. Assure that
    these standards address all known security vulnerabilities and are
    consistent with industry-accepted system hardening standards.

The United States Defense Information Systems Agency (DISA) publishes sets of
security hardening guides called `Security Technical Implementation Guides
(STIGs)`_.  They're comprehensive and they provide mechanisms for checking
secured systems for compliance with the standards.  In addition, they are in
the public domain.

.. _Security Technical Implementation Guides (STIGs): http://iase.disa.mil/stigs/Pages/index.aspx

Proposed change
===============

The proposed changes include:

#. Create a new role in a new repo to hold the security tasks
#. Write documentation about the hardening standards applied

   * Is this standard already deployed by default in Ubuntu 14.04 or by OSAD
     already?
   * If a standard is applied, what does a deployer gain from it?
   * If a standard is skipped, why was it skipped and what does the deployer
     lose?

#. Submit patches that actually apply those hardening standards

   * Start by making a bug for each with a description of what will be changed
     and why
   * Determine whether the patch belongs in openstack-ansible or within a new
     security-hardening role that can be pulled into openstack-ansible during
     deployments

#. Create an automated way to test that the security changes are applied and
   they don't cause negative impacts on openstack-ansible deployments

   * This could be done via OpenSCAP or via CIS' Java-based checker
   * Needs to be checked via gate check jobs

#. Make it easy for deployers to import the security hardening role into
   openstack-ansible

   * Should be easily pulled into an openstack-ansible deployment if a deployer
     chooses

Here are several examples of security improvements recommended by the RHEL 6
STIG which apply well to Ubuntu:

* V-38497: The system must not have accounts configured with blank or null
  passwords.
* V-38476: Vendor-provided cryptographic certificates must be installed to
  verify the integrity of system software.
* V-38607: The SSH daemon must be configured to use only the SSHv2 protocol.
* V-38614: The SSH daemon must not allow authentication using an empty
  password.
* V-38673: The operating system must ensure unauthorized, security-relevant
  configuration changes detected are tracked.
* V-38632: The operating system must produce audit records containing
  sufficient information to establish what type of events occurred.

Alternatives
------------

No known alternatives.


Playbook/Role impact
--------------------

Depending on the nature of the change and the usefulness to deployers, the
changes may be applied directly to existing roles in openstack-ansible or they
may be applied to security hardening role that is optionally pulled in during
openstack-ansible deployments

Any changes which could affect the performance, stability, or functionality of
a production deployment would be disabled by default and heavily documented.
Deployers could then make an educated decision on whether or not they want that
security hardening standard enabled.

Upgrade impact
--------------

If security features are added via feature flags and disabled by default, the
effect on upgrades would be very minimal if they're even noticed at all.  All
configuration changes should be examined individually to determine if they will
have an impact on upgrades.


Security impact
---------------

The entire goal of this spec is to have a positive security impact without
becoming an operational burden.


Performance impact
------------------

It's possible that some security changes could impact the performance of a
running OpenStack system.  As noted in *Upgrade impact* above, these
configuration changes would need to be examined individually to determine the
balance between security and performance impacts.


End user impact
---------------

End users shouldn't notice the majority of the security changes.  They will
still interact with API endpoints and virtual machines as they do today.
There's a chance that some security improvements could impact an end user, but
deployers will have full control of how those improvements are applied.


Deployer impact
---------------

Deployers could potentially be able to build OpenStack systems that are more
secure by default.  However, if these security features are disabled by
default, we need solid documentation that tells users how to enable these
features and what the impact of enabling those features might be.

Deployers would need to explicitly include the security hardening role within
their openstack-ansible deployments.


Developer impact
----------------

Developers would need to include the security hardening role within their
deployments if they wanted to test openstack-ansible with additional security
enhancements.


Dependencies
------------

This spec has no dependencies.


Implementation
==============

Assignee(s)
-----------

Who is leading the writing of the code? Or is this a blueprint where you're
throwing it out there to see who picks it up?

If more than one person is working on the implementation, please designate the
primary author and contact.

Primary assignee:
  Major Hayden (LP: rackerhacker, IRC: mhayden)

Other contributors:
  Cody Bunch (LP: <TBA>, IRC: <TBA>)

Work items
----------

The work items are in the *Proposed change* section above in a numbered list.
Documentation should come first, followed by actual configuration changes.


Testing
=======

The usual gate checks can be used for these changes.  Also, each individual
commit can be tested individually.


Documentation impact
====================

Documentation is a critical piece of this spec, and it's the first step in the
process.  It would be helpful to get the documentation team to weigh in on some
of the documentation changes to ensure it makes sense for deployers.


References
==========

Mailing list thread:

* http://lists.openstack.org/pipermail/openstack-dev/2015-September/074104.html

IRC discussion:

* http://bit.ly/1F1wBgB

DISA STIGs:

* https://en.wikipedia.org/wiki/Payment_Card_Industry_Data_Security_Standard
