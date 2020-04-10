Documentation improvements
##########################
:date: 2018-01-08 22:00
:tags: docs, user stories, walkthrough

Include the URL of your launchpad blueprint:
  * https://blueprints.launchpad.net/openstack-ansible/+spec/example

People are often confused when they deploy with openstack-ansible,
because they only partially read the documentation, or landed on
the wrong documentation.

Our deployment guide is already close to what I call a wizard/
walkthrough, but some parts are easily missed by the deployers.

On top of that, some very nice advanced documentation are often skipped, or
aren't promoted to their right value.

Problem description
===================

When people land on our deployment guide, which is probably the first
link they access, whether they come from the OpenStack deployment guides
https://docs.openstack.org/pike/deploy/ or from our main developer page
https://docs.openstack.org/openstack-ansible/latest/ , they are facing
the following issues:

* The landing page is overwhelming, as its a series of link. What do you click?
* The first links clicked (example: https://docs.openstack.org/project-deploy-guide/openstack-ansible/pike/overview.html#)
  is just more clicks towards content, and doesn't provide any useful information (the structure is already displayed on the left side of the page).
* Anyone wanting to quickly deploy an openstack-ansible cloud has no way
  to know we have an AIO toolkit that could help.
* Anyone wanting to deploy a production cluster with Netapp for example
  will most likely not find the appropriate documentation while reading
  the deploy guide: It's easy to miss the importance of configuration
  on https://docs.openstack.org/project-deploy-guide/openstack-ansible/latest/configure.html#advanced-service-configuration
* The AIO is listed in the "contributor" guides, where it could be available
  on any deployment part.
* There is an hard to find "advanced configuration" section,
  hidden inside the operations guide.
  It should probably be an appendix of the deploy guide.
* We have many overlaps of documentation doing about the same thing,
  we should clean things up (for example the advanced configurations
  in deploy + operations, the inventory in operations + contributors +
  reference)
* There are too many appendices in the deploy guide. Some deserve their
  own section. According to the spec http://specs.openstack.org/openstack/docs-specs/specs/pike/os-manuals-migration.html,
  "end-user content such as concept guides, advice, tutorials, step-by-step instructions for using the CLI to perform specific tasks, etc."
  I think we could technically move scenarios there, as they are step-by-step instructions using shell scripts to perform some specific deploys.
* The role maturity is hard to find. If I were a new deployer, I'd like
  to know what I could do with OpenStack-Ansible, and the role maturity
  matrix would tremendously help. Sadly I'd never see it in my first
  read of the documentation.
* The upgrade guide is our user guide. Why not considering upgrades as
  a specific kind of operation? In my opinion, it should be part of
  the operations guide, as a major chapter in the operations, in the same
  way as minor updates, or scaling the environment.
* We don't motivate people to contribute back directly from the
  deploy guide. The last step after verifying that the system works
  should be how to extend and contribute to OpenStack-Ansible.
* People could be confused on how to best contribute to the project.

Proposed change
===============

Have some kind of notice at the beginning of the deploy guide,
pointing to our user stories (but advising to read the deploy guide
first). The first user story would be the AIO, with the quickstart
AIO content, for those who want devstack-like easiness, developers,
or for those who want to prototype.
Add more user stories into the user guide, for ceph (test, prod and
ceph-ansible integration), for l3 routed scenarios (tests and prod),
for offline installs.

Move advanced topics like inventory, container networking, custom
layouts and security principles into a new "reference" section of
the documentation. This section should probably hold the links
to roles' documentations, and should also be linked from the deploy
guide where appropriate.

Highlight the importance, in the deploy guide, of our advanced
topics (reference). It's important for new deployers to know
where to find documentation on how to do X that's not part of
a user guide.

At the end of the deploy guide, continue the deploy story by
pointing to our operations and contributions
guide. That could be added into a next steps section.

The contributors' guide can also be enhanced by listing where
the help is wanted: docs (and their manually testing, like for the
operational guide), bugs (triaging and fixing the low hanging
fruits), test coverage, ... This section could be altered when
the priorities change.

For improving the reading experience, ensure that each page has
a proper structure:

- Only content should appear in the content part of the page
- The chapters should only be in the upper-left section of
  the page ToC, and pointing to this guide chapters, not the whole
  documentation items
  (avoiding something like https://docs.openstack.org/openstack-ansible/latest/user/index.html)
- The page headers should only be in the lower-left section of the
  page ToC.


Alternatives
------------

Not changing the docs, or partially implementing those changes.

Playbook/Role impact
--------------------

None

Upgrade impact
--------------

None


Security impact
---------------

None

Performance impact
------------------

None

End user impact
---------------

None


Deployer impact
---------------

New deployers should be less overwhelmed by Openstack-Ansible

Developer impact
----------------

None

Dependencies
------------

None

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  jean-philippe-evrard

Other contributors:
  * TODO

Work items
----------

Each paragraph of the proposed change can be considered as a work item.

Testing
=======

Nothing new.

Documentation impact
====================

This is a docs only change, so this whole change has a documentation impact.
However, because we don't change the structure of the docs themselves,
it should not be very difficult to implement.

References
==========

This improvements only happen to improve our readability, and to follow
what's generally expected to find in each of the documentations:


