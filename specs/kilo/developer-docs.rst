openstack-ansible overview documentation
########################################
:date: 2015-04-13 13:00
:tags: documentation, docs, developer

Currently, the openstack-ansible repository does not have any cohesive
developer documentation. This proposal aims to begin such documentation,
providing a overview and reference for new contributors.

Documentation covered in this proposal is *not* intended to be exhaustive.

Also, documentation is a constantly shifting thing - this proposal is just
intended to get the initial documents created; documentation maintenance
falls outside of scope. Reviewers should help make sure patches adequately
update associated documentation.

A blueprint for this proposal can be found at:

https://blueprints.launchpad.net/openstack-ansible/+spec/developer-docs

Problem description
===================

Currently, when new contributors come to the repository, there isn't much to
help them to understand the general structure of the project. Currently, there
is the development-stack.rst file, which provides a very brief introduction
to getting an all-in-one (AIO) install started, as well as tearing down
an environment, but there's not much else. This can be intimidating for
newcomers, as well as current contributors who might forget details of
some portion of the large code base.

Proposed change
===============

This proposal recommends making a new docs repo, which would contain Sphinx
documentation on the following documentation:

* Overview of doing deployments using openstack-ansible

* Variable files, for knowing which variable files are used where in the process.

* Scripts. This section will cover using bootstrap, gating, and teardown
  scripts. It might also document some of the important variables/parameters
  for these scripts. The openstack-ansible wrapper would be nice to cover
  here and in the extending section.

* Playbooks should document the high-level playbooks that prepare physical
  hosts, create containers, and install OpenStack.

* Repository role/playbook. Since the repository is fairly unique to
  openstack-ansible, this should be probably be a bit more detailed than
  the rest of the of the documentation. The openstack_services.yml and
  openstack_other.yml files are of particular interest here.

* Inventory management. This section should discuss the dynamic_inventory.py
  file and and the inventory_management.py files.

* Extending openstack-ansible. This would cover using the conf.d and env.d
  directories, as well as user_extras.yml files. Changes to ansible.cfg
  necessary might be useful, too.

Also, the docs directory should be built by Sphinx on a regular basis,
preferably at the gate.

Some topics are explicitly out of scope for this changes. In particular:

* Host networking setup. This is highly individualized per environment, and
  too broad to cover here.

* Individual roles. The individual roles should not be documented as part of
  this. There are too many roles and too many changes to be able to keep up with
  those changes at the openstack-ansible level.

* End user documentation. For this specification, the 'end user' is a deployer
  or user of the deployed OpenStack system. The installation guide, operations
  guide, and user/admin guides are all out of scope.

Alternatives
------------

We could not do this documentation, leaving the repository as is.

Playbook impact
---------------

There should be no impact on the playbooks; this change only adds files
outside of the playbooks.

Upgrade impact
--------------

This documentation will have to be kept up-to-date with releases. Since
documentation is an on-going process, it falls to reviewers to enforce
documentation updates to playbook changes.

Security impact
---------------

Since this change is not to the playbooks or scripts, it should have no
security impact.

Performance impact
------------------

This will not have a production performance impact. I do propose adding a
docs build job to the gating process, which would extend gating job times
by some unknown amount.

End user impact
---------------

Users of a deployed cloud would likely never see this change. It's largely
targeted at developers of this project or deployers.

Deployer impact
---------------

There should be little to no deployer impact. This documentation will be
for developers, mostly, but deployers may be able to use it as reference
for running scripts and tools on their deployment.

Developer impact
----------------

This documentation will be targeted mostly at developers in the hope that it
will be easier for new contributors to understand how the project works and
where to start. This can also be useful as a reference for existing developers.

The Sphinx build process may add some overhead, since developers should build
the documentation before pushing their changes.

Where this proposal differs from the CONTRIBUTING.txt is the focus -
CONTRIBUTING.txt is largely about the process around getting changes into the
codebase. In contrast, the docs directory should cover technical information
about how to use the repository.

Dependencies
------------

This change does not depend on any other blueprints or specs. It can be done
largely in parallel with other projects and issues.

Implementation
==============

As described earlier, this will be implemented with ReST files in a docs
directory at the root of the repo. Also, there will be a dependency on
Sphinx in the dev-requirements, and a script added to run the Sphinx docs
build job a tthe gate.

Assignee(s)
-----------

Other contributors are welcome to work on the mentioned sections.


Primary assignee:
    nolan-brubaker **palendae**
Other contributors:
  <launchpad-id or None>

Work items
----------

* Add the docs directory and some basic structure files, like an index page
  and a Sphinx configuration file.

* Add a file for each section to the docs directory, as well as to the index
  page.

* Add a Sphinx build job to the gating scripts that only runs if there was a
  change to the docs directory.


Testing
=======

This change will add a Sphinx build job to the gating process. The Sphinx
build job should not run on changes that have no affected docs files.

The Sphinx documentation build job should succeed for the change to merge.


Documentation impact
====================

As mentioned above, this will create a new docs repo that the docs team
can then build more detailed documentation in reference to.


References
==========
N/A
