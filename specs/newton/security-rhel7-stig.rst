Apply RHEL 7 STIG hardening standard
####################################
:date: 2016-08-11 00:00
:tags: security

The Security Technical Implementation Guide (STIG) for Red Hat Enterprise Linux
(RHEL) 7 is in the final stages of release. The security hardening role needs
to be updated to apply these new requirements to Ubuntu 16.04, CentOS 7 and
RHEL 7.

* https://blueprints.launchpad.net/openstack-ansible/+spec/security-rhel7-stig

Problem description
===================

Today, the openstack-ansible-security role uses the RHEL 6 STIG as the basis
for all of the security configurations applied to Ubuntu 14.04, Ubuntu 16.04,
CentOS 7, and RHEL 7.  However, the new RHEL 7 STIG is in the final stages of
its release and the new security configurations provide a stronger security
posture for all systemd-based distributions, including:

* Ubuntu 16.04
* CentOS 7
* RHEL 7

There are some challenges with a wholesale change to the RHEL 7 STIG:

* It doesn't apply well to Ubuntu 14.04
* It uses a new numbering scheme which doesn't match with the RHEL 6 STIG
* Many tasks have no overlap with the RHEL 6 STIG

Proposed change
===============

The current role structure is flat and the differences between the
distributions are handled within each task YAML file. The proposed new layout
would look something like this:

.. code-block:: text

   /main.yml
   /rhel6stig/main.yml
   /rhel6stig/auth.yml
   /rhel6stig/boot.yml
   /rhel6stig/...
   /rhel7stig/main.yml
   /rhel7stig/auth.yml
   /rhel7stig/boot.yml
   /rhel7stig/...

The root ``main.yml`` would have a ``when:`` that would include the correct
``main.yml`` from the STIG version subdirectories.  This comes with some nice
benefits:

#. This would ensure that the functionality for Ubuntu 14.04 is unchanged.

#. When support for Ubuntu 14.04 is no longer needed, it could easily be
   removed later by simply removing the ``rheli6stig`` directory and the
   corresponding ``include:`` from the root ``main.yml`` file.

#. Some of the existing clutter in the role could be removed since Ubuntu
   16.04, CentOS 7 and RHEL 7 are closely aligned (because they all use
   systemd).

Alternatives
------------

Switch the entire role to the RHEL 7 and drop Ubuntu 14.04 support.
  This could be upsetting for existing users of the role on 14.04.

Interleave the RHEL6/RHEL7 STIG configurations in the existing role
  This could lead to lots of clutter and could add difficulty when Ubuntu 14.04
  support needs to be removed.

Create a new role
  Deployers could be confused by a new role and it would require changes to
  OpenStack-Ansible's integrated build.

Playbook/Role impact
--------------------

See the **Proposed change** section above for details.

Upgrade impact
--------------

If a deployer is running the Newton release of the role on Ubuntu 16.04,
CentOS 7, or RHEL 7, they will notice lots of additional security
configurations being applied by the role per the requirements of the RHEL 7
STIG. Backing out security configurations from the previous versions of the
role shouldn't be necessary.

Security impact
---------------

This change will improve the role's capability to secure new systemd-based
distributions, such as Ubuntu 16.04, CentOS 7, and RHEL 7.

Performance impact
------------------

As with the previous versions of the role, the updates to the role from the
RHEL 7 STIG should not cause performance impacts or downtime on the system.

End user impact
---------------

End users should not notice a difference when these changes are made.

Deployer impact
---------------

Deployers will apply the role using the same commands as they do now.  However,
they will see some new changes:

* New configurations being applied that weren't being applied previously
* New variables for controlling the security configurations in the RHEL 7 STIG

Developer impact
----------------

Developers must ensure that RHEL 7 STIG content is kept separate from RHEL 6
content.  This will be documented within the tasks themselves as well as in
the formal role documentation.

Dependencies
------------

This change has no dependencies.

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  Major Hayden (LP: rackerhacker, IRC: mhayden)

Work items
----------

#. Create a directory for the RHEL 7 STIG content and begin adding tasks there
   to apply security configurations.

#. Update documentation to reflect the new configurations and any new variables
   which exist to configure the role's actions.

#. When the RHEL 7 STIG content is working well on Ubuntu 16.04, CentOS 7, and
   RHEL 7, the root ``main.yml`` should include the tasks from the RHEL 7 STIG
   directory.

#. At a later date, Ubuntu 14.04 support could be removed by deleting the RHEL
   6 directory, removing unneeded variables, and removing unneeded
   documentation.

Testing
=======

The OpenStack CI environment would test the security role in the same way that
it does now.  Testing could be adjusted during the first phase of RHEL 7 STIG
development so that both pathways (RHEL 6 STIG and RHEL 7 STIG) are tested on
Ubuntu 16.04 and CentOS 7.

RHEL 7 testing will need to be manual since OpenStack CI has no RHEL image.

Documentation impact
====================

New documentation will be needed for the RHEL 7 STIG security configurations as
well as any new variables that are introduced. This will need to be done
carefully (perhaps in a draft directory) until the RHEL 7 STIG content is ready
to be applied to Ubuntu 16.04 and CentOS 7.

References
==========

* DISA STIGs: http://iase.disa.mil/stigs/os/unix-linux/Pages/index.aspx

* openstack-dev mail: http://lists.openstack.org/pipermail/openstack-dev/2016-August/100883.html
