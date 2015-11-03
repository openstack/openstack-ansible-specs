IRR - rsyslog client
####################
:date: 2015-11-01
:tags: independent-role-repositories, rsyslog_client

Split out the rsyslog client role into it's own repository.


Problem description
===================

Roles are all contained within a single monolithic repository making it
impossible/difficult to consume the OSA roles outside of deploying the
entire stack.


Proposed change
===============

To ensure that the OSA project is consumable by other stacks using different
architectures, deployment methods, and capabilities the role
"rsyslog_client" need to be moved from the monolithic stack and
into the it's own role repository.


Alternatives
------------

Leave everything the way it is. However doing that will hurt general OSA
adoption.


Playbook/Role impact
--------------------

* No impact to the playbooks.
* The role will be removed from the main stack. The plugins, filters, and
  libraries may need to be locally updated.


Upgrade impact
--------------

While the change will impact the placement of the role it will not impact
upgrade-ability of the stack. The general workflow will need to be updated
to ensure that users are updating roles on upgrade  using the Ansible
galaxy interface however generally speaking this is already being done for
the deployer when running the ``bootstrap-ansible.sh`` script.


Security impact
---------------

n/a


Performance impact
------------------

Moving the role to an external repository will cause an impact in time  to
role resolution however that impact should be minimal.


End user impact
---------------

n/a


Deployer impact
---------------

Deployers will need to be aware of the new role locations and how to update
existing roles however this should be minimal considering the tooling for
updating existing roles already exists


Developer impact
----------------

Developers will need focus work within the roles which will exist within
separate repositories.


Dependencies
------------

n/a


Implementation
==============

Assignee(s)
-----------

Primary assignee:
  https://launchpad.net/~kevin-carter (IRC: cloudnull)


Work items
----------

* With the role moved, tests will be created within the role via OpenStack CI
  to ensure that the role is performing the actions that its supposed to.
* Updated documentation via the "README.rst" will be created to show how the
  role can be used standalone.
* Example local inventory will be created to show how the role can be used.
  The local only inventory will also be used for testing the role.


Testing
=======

* The test cases will deploy the role into a regular DSVM image
* The role will execute itself locally
* Once the role has completed an Ansible test play will run through several
  assert tasks to ensure the role functioned as intended.


Documentation impact
====================

The base README.rst file will be updated to explain how the role can be used
as a standalone role.


References
==========

n/a

