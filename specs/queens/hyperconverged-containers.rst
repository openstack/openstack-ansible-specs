Hyper-Converge Containers
#########################
:date: 2017-09-01 22:00
:tags: containers, hyperconverged, performance

Reduce container counts across the infra structure hosts.

To lower our deployment times and resource consumption across the board. This
spec looks to remove single purpose containers that have little to no benefit
on the architecture at scale.

This change groups services resulting in fewer containers. This does not
mix service categories so there's no worry of cross polluting a different
service with unknown packages or unknown workloads. We're only look to minimize
the container types we have and simplify operations. By converging containers
we're removing no less than 10 steps in the container deployment process and the
service setup. Operationally we're reducing the load on operations teams
managing clouds at any scale.


Problem description
===================

When we started this project we started with the best of intentions to create a
pseudo micro-service model for our system layout and container orchestration.
While this works today, it does create a lot of unnecessary containers in terms
of resource utilization.


Proposed change
===============

Converge groups of containers found within the `env.d` directory into a single
container where at all possible. Most the changes we need to get this work done
have already been committed. In some instances we will need to "revert a change"
to get the core functionality of this spec into master but there will be little
to no development required to get the initial convergence work completed.

Once the convergence work is complete we intend to develop a set of playbooks
which will allow the deployer to run an "opt-in" set of tasks which will cleanup
containers and services wherever necessary. Services behind a load balanacer
will need to be updated. Updates to the load balancer will be covered by the
"opt-in" playbooks provided the environment is using our supported software
LB (HAProxy). The "opt-in" playbooks will need to be codified, tested, and
documented. Should it be decided that the hyperconverged work is to be
cherry-picked to a stable branch, the new playbooks will need to first exist
and be tested within our periodic gates. We should expect no playbook impact
in-terms of the general deployer workflow.


Alternatives
------------

We could leave everything as-is which carries the resource requirements we
currently have along with an understanding that the resources required will
grow given the fact OpenStack services, both existing and net new, are ever
expanding.


Playbook/Role impact
--------------------

At least one new playbook will be added allowing a deployer to cleanup old
container types from the run-time and inventory should they decide to. The
cleanup playbook(s) will be "opt-in" and will not be part of our normal
automated deployment process.


Upgrade impact
--------------

There is no upgrade impact with this change as any existing deployment would
already have the all required associations within inventory. Services would
continue to function normally after this change. Greenfield deployments on the
other hand would have fewer containers to manage which reduces the resource
requirements while also ensuring we retain the host, network, and process
separation we have today.

We will create a set of playbooks to cleanup some of the redundant containers
that would exist post upgrade however the execution of this playbook would be
opt-in.


Security impact
---------------

Security is not a concern within this spec however reducing the container
count would reduce the potential attack surface we already have.


Performance impact
------------------

Hyperconverging containers will reduce resource consumption on physical host.
Reducing the resources required to run an OpenStack cloud will improve the
performance of the playbooks and the system as a whole.


End user impact
---------------

N/A


Deployer impact
---------------

Deployers will have fewer containers to manage and be concerned with as they
run clouds for long periods of time.

* Within an upgrade scenario a deployer will have the option to "opt-in" to a
  hyperconverged setup. This change will have no service impact on running
  deployments by default.


Developer impact
----------------

N/A


Dependencies
------------

* If we're to test the "opt-in" cleanup playbooks we'll need a periodic upgrade
  gate job. The playbooks would be executed by the upgrade gate job and post
  results to the ML/channel so that the OSA development team is notified of the
  failure.


Implementation
==============

Assignee(s)
-----------

Primary assignee:
  Kevin Carter (IRC: cloudnull)
  Major Hayden (IRC: mhayden)


Work items
----------

* Converge the containers into fewer groups
* Create the "opt-in" container reduction playbooks
* Document the new playbooks


Testing
=======

* The core functionality of this patch will be tested on every commit.
* If the upgrade test dependencies are met we can create a code path within the
  periodic gates and test the "opt-in" cleanup playbooks.


Documentation impact
====================

Documentation will be created for the "opt-in" container cleanup playbooks
created.


References
==========

N/A
