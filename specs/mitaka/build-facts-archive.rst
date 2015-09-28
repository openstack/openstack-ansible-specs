Build Facts Archive
###################
:date: 2015-04-23
:tags: archive, deployment, information

Create a script to archive all valuable information about a deployment.
This information includes but is not limited to the following: kernel version
of all physical host, version of OSAD that is currently installed, all
installed packages and their versions, all running containers and their
installed packages, latest tempest test run, all relevant OSAD configuration
files (openstack_user_config, etc), host networking configuration,
host disk configuration.

* https://blueprints.launchpad.net/openstack-ansible/+spec/build-facts-archive

Problem description
===================

Currently there is no simple way to get information about a deployment. The
current process requires a log into the deployment host and then knowledge of
ansible and the openstack_inventory.json file and its groups to correctly
structure a ansible query to gather information.

It is also challenging to create a tool outside of OSAD to do some automation
around aggregation of of deployment information as you need to parse the
inventory file or know exactly which host or container you need to access to
get information.

Proposed change
===============

A simple script that the user can run to gather all predetermined important
information to give a solid top down view of a deployed cluster.


Alternatives
------------

This script could live in the rpc-extras repository instead of OSAD. This
would not be ideal as it would only help the users who are using rpc-extras.
If it resides in OSAD then all users get a simple way of getting a quick top
down view of what their current cluster has.

Playbook impact
---------------

There will need to most likely be a playbook added to accomplish the task of
gathering valuable information from each host / container based on their role
/ group. This playbook will not be deployment impacting.

Upgrade impact
--------------

None

Security impact
---------------

This could potentially touch all containers to gather secure information such
as: configuration files which may contain passwords, information about
keystone users (names, roles, etc). This is a minimal risk as the user would
have to export the output off the host. If someone is running this script they
already have access to this information as they are logged onto the deployment
host.

Performance impact
------------------

None

End user impact
---------------

None

Deployer impact
---------------

This change will give the deployer an easy way to gather current information
about their cloud. This could help troubleshoot config problems as well as
allow them a quick insight into their latest test results. This will even
allow them to see package discrepencies and help them prepare for an upgrade.


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

Open to all

Primary assignee:
  None

Other contributors:
  None

Work items
----------

Create a script to:
    - create the archive directory
    - gather all relevant deployment information
    - tarball archive directory
    - move tarball to well known location
    - remove archive directory

It would be up to the end user / deployer what they do with the tarball, but
it should be placed in a resonable spot on the deployment host that would be
easy to find / access for the deployer.

Testing
=======

This should add a task to gating/commit/nightly to run this script and
return the captured archive tarball as a jenkins artifact. Tempest results
could also be sent to jenkins so that the results.xml can be displayed.
This should help developers / qe see testing trends and allow the users
of jenkins to more accurately find bugs in a more timely manner.


Documentation impact
====================

A simple reference to this script in the user guide would be all that is
needed if it is determined that it warrents it.


References
==========

If you look at the current scripts located in the scripts directory

* https://github.com/openstack/openstack-ansible/blob/master/scripts/scripts-library.sh#L226-L279

a lot of this information is already gathered about the host that the script
is run on. This proposal should use the information that is gathered as a
blueprint to some of the information that should be gathered about all hosts.
