Create Operations Guide
#######################
:date: 2016-12-19 17:00
:tags: docs, ops

Blueprint: Create OpenStack-Ansible Operations Guide
* https://blueprints.launchpad.net/openstack-ansible/+spec/create-ops-guide

This specification proposes the development of an OpenStack-Ansible Operations
Guide for the Ocata release.

Problem description
===================

During the Newton development cycle, the Installation Guide was revised which
focused on providing a method for installing OpenStack for a test
environment and production environment. As noted in the
`Installation Guide spec <https://review.openstack.org/#/c/323471/12/specs/newton/osa-install-guide-overhaul.rst>`_,
the operations content did not belong in the Installation Guide as it
reduced the user's focus to install OpenStack, and was temporarily relocated to
the following Developer Documentation pages:

* http://docs.openstack.org/developer/openstack-ansible/developer-docs/ops.html
* http://docs.openstack.org/developer/openstack-ansible/developer-docs/extending.html

There is a need to develop a standalone Openstack-Ansible operations
guide that will address an operator's need for information on managing and
configuring an OpenStack cloud using OpenStack-Ansible.

Proposed change
===============

The main focus of the operations guide is to re-organise the current content and
develop new content so an OpenStack operator can easily search for information
on maintaining their environment, troubleshooting, and resolving issues.

The proposed changes are:

* A new ToC with input from developers and operations: https://review.openstack.org/#/c/409854/
* Removal of duplicated content from the OpenStack manuals operations guide
  (so that this guide focuses primarily upon OpenStack-Ansible operations).
* Structuring the guide in a 'runbook' format for the following reasons:

  #. Ensuring the guide includes lower-level how-to's for anyone starting to
     operate their own cloud.

  #. Ensuring the guide includes higher-level troubleshooting information for
     more experienced operator.

  #. It is structured to make it easy for operators to find the information
     they are looking for.

* Review and update current operations content to follow the
  openstack-manuals documentation conventions.

Alternatives
------------

* The current operations content and any future content will remain in the
  Developer Documentation.

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

These changes will improve the end user experience, by providing
a more structured and better flow of information to operate your OpenStack
cloud.

Deployer impact
---------------

N/A


Developer impact
----------------

N/A


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
  Andy McCrae (andymccr), OpenStack-Ansible PTL
  Darren Chan (darrenc)
  Robb Romans (rromans)

Work items
----------

- Clarify and obtain consensus on the content structure
- Gather information from SMEs as needed
- Create a draft directory for operations guide changes
- Create a work items list and allocate resources
- Ensure documentation meets openstack-manuals writing conventions
- Test draft documentation before publication

Testing
=======

The testing will be conducted by the community once a draft is available.
OpenStack-Ansible users will be asked to utilise the new operations guide
to perform the OpenStack operations and evaluate if the information provided
is accurate, clear, and concise.

Documentation impact
====================

This is a documentation change, N/A.

References
==========

* ToC planning

  * https://docs.google.com/document/d/1xeJ_lep7P2e7HLbRFG57Dx4W9s8brkuNIqJmOvheWKI/edit?usp=sharing

  * https://review.openstack.org/#/c/409854/
