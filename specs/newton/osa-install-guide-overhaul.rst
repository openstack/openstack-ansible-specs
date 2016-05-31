Overhaul of the current OpenStack-Ansible Installation Guide
############################################################
:date: 2016-05-31 00:00
:tags: docs

Blueprint: Overhaul of the current OpenStack-Ansible Installation Guide
  * https://blueprints.launchpad.net/openstack-ansible/+spec/install-guide
  * https://blueprints.launchpad.net/openstack-ansible/+spec/osa-install-guide-overhaul

After the 2016 Austin summit, there was a discussion and a consensus
surrounding the current state of the OpenStack-Ansible Installation Guide.

.. note::

    A `blueprint and spec <https://review.openstack.org/#/c/241037/1>`_ were previously created
    with the intention of improving the documentation that pushed the summit discussion.

Currently, the OpenStack-Ansible install guide has minimal installation
information, and a lot of configuration information. This specification proposes
a more formalized plan to separate this information and streamline the
installation guide to make it easier and quicker to install OpenStack.

Problem description
===================

The OpenStack-Ansible Installation Guide contains information that does not
necessarily pertain to that of an installation guide structure. It has
accumulated a lot of configuration information and reference information that
reduces the user's focus and simplicity to install OpenStack.

The current installation guide also does not follow the openstack-manuals
documentation conventions.

Proposed change
===============

The main focus of the installation guide is reorganising and developing
content so a deployer makes very few decisions and minimal configuration
to deploy an OpenStack test environment and production environment.

The proposed changes are:
* Clearly define reference architecture and develop use case configuration
  examples in an appendix.
* Removal of the configuration information from the current installation guide
  and including it in the OpenStack-Ansible role documentation.
* Migrate operations content temporarily to openstack-ansible-ops repo
  until an operations guide can be produced.
* Restructure the guide to include basic deployment configuration.
* Appendices that include configuration file examples, neutron plugins,
  cinder options and additional resources relevant to an OpenStack-Ansible
  installation.
* Include links to role based documentation from the Installation Guide.

Alternatives
------------

* Leaving the installation guide as is, and migrating only the configuration
information to the developer docs.

* Consider revising the installation guide to meet criteria in `project-specific installation guide
<http://specs.openstack.org/openstack/docs-specs/specs/newton/project-specific-installguides.html>`_
and publish to docs.openstack.org

Playbook/Role impact
--------------------

N/A

Upgrade impact
--------------

N/A

Security impact
---------------

N/A

Performance impact
------------------

N/A

End user impact
---------------

These changes will hopefully improve the end user experience, by providing
a more structured and better flow of information to install OpenStack.

Deployer impact
---------------

N/A


Developer impact
----------------

Move existing content over to the roles first, then developers must
submit any new documentation to the role repositories.

Dependencies
------------

N/A

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  Alexandra Settle (asettle)

Other contributors:
  Darren Chan (darrenc), Jesse Pretorius (odyssey4me),
  Travis Truman (automagically), Major Hayden (mhayden)

Work items
----------

- Clarify and obtain consensus on the content structure
- Gather information from SMEs as needed
- Create a draft directory for installation guide changes
- Create a work items list and allocate resources
- Ensure documentation meets openstack-manuals writing conventions
- Test draft documentation before publication

Testing
=======

The testing will be conducted by the community once a draft is available.
OpenStack-Ansible users will be asked to follow the new installation guide
to install OpenStack and evaluate if the information provided is accurate,
clear, and concise.

Documentation impact
====================

This is a documentation change, N/A.

References
==========

* Design Summit discussion:
  `https://etherpad.openstack.org/p/openstack-ansible-newton-role-docs`_

* ToC planning:
  `https://docs.google.com/document/d/1WdcA7jIp8w1C52pJu4JmympFe8cOvcxi1I2E19Y6XYE/edit?ts=5743fe3f#heading=h.jg8guj3uzhzw`_