Refactoring OSA inventory
#########################
:date: 2018-04-12 22:00
:tags: osa, inventory

The inventory as it stands today has been growing in complexity
and has only grown organically since its first implementation
in icehouse. Given that Ansible has changed a lot and has added
capabilities which were not available in those early versions,
it is time to take a step back and look at how it can be re-worked
to reduce technical debt and make it easier to maintain.

Problem description
===================

The current OpenStack-Ansible inventory provides the following
features:

* Assignment of hosts into groups
* Generating the group structure
* Assigning host variables
* Generating container inventory_hostnames
* Assigning and tracking container IPs based on cidr_networks,
  reserved IPs, and already allocated IPs.

All these features are included into a single dynamic inventory script,
because at the time of its creation, only one inventory was allowed
at a time in an ansible cli call.

The dynamic inventory shipped by OSA is core of the functionality of
OpenStack-Ansible, yet it is not well understood, neither by the core
maintainers nor by new contributors.

As a result, the inventory has grown organically, both in code and
in memory usage (changes in the way we deploy things, adding new
groups, adding edge cases), and has not seen much maintenance
to reduce its scope or the technical debt.

At this point, due to a lack of tests and the complexity of the code,
it is difficult to work on without causing hidden breakages
which are often only found months later. Adding tests is
unrealisticly hard for this legacy code.

The problems can therefore be summarized in a few points:

* The inventory needs to be cleaned up of unnecessary groups and
  assignments, but it is difficult to clean up effectively
  without causing hidden breakages.
* We have to carry code in openstack-ansible that is not actively
  maintained
* We have to execute code that's not actively audited, while
  it would be technically possible to avoid the execution of
  code with very few limitations for the end-user.
* Introducing tests to verify regressions was attempted during
  the Newton, Ocata and Pike development cycles - but that
  has done nothing more than increase the code complexity
  and has done nothing to improve the reliability.

Proposed change
===============

Now that we are using Ansible 2.4, we can:

* Stack inventories together, and therefore we can split inventories
  into smaller inventories if necessary
* Import, and convert inventories to a more readable format.

What I am proposing is to use static files for inventory.
It is easier for people to edit the inventory, and review it.
It's easier to manipulate, and doesn't require our code to
run or edit it.

Host vars, group vars, and inventory structure would be
static files, and slimmed down to the minimum.

Here are two example of slimming down (hosts vars, and inventory):

* For me, the features to track proper IP assignment is the
  scope of a CMDB/IPAM. We shouldn't reinvent the wheel there.
  Instead this should be spun out of the inventory.
  People should either:

  * use the old inventory to keep the same features, but
    we add a warning that the code is deprecated
  * provide their own IP addresses in a static file
  * provide their own dynamic inventory script or use a lookup
    to fetch data from their IPAM.

  With the generation of IPs outside the scope of the inventory,
  we could simplify the dynamic inventory further.

* For me, the groups like haproxy, haproxy_all, haproxy_hosts
  or haproxy_containers are all confusing. Some are used
  interchangeably, which led to bugs. The proliferation of
  groups is only due to our inventory.
  These can all be consolidated into a single
  group, by changing the playbooks and roles. This is
  not only restricted to haproxy, and this pattern of
  group reduction should be extended to all our inventory.

So, at first we need to keep the same configuration style
(conf.d/env.d/openstack_user_config). The generated json
would then go through a script to generate and clean
the static files.

That script would be part of the deploy and upgrade
process.

Later, we could re-think the conf.d/env.d/openstack_user_config,
or keep it the same but completely change the underlying code.
That wouldn't be a problems, because it could be done on the
side, as a different inventory system. We would have, on the
way, documented the input and outputs of the inventory,
which could then be used for building test cases.

Alternatives
------------

Do nothing

Playbook/Role impact
--------------------

Removing references to old inventory data like old groups.
Use lookups or ansible_facts better to reduce the amount of hostvars.

Upgrade impact
--------------

Because our inventories are already in a bad state, we already have
hosts in the wrong groups.

Upgrade would need to run the tool to migrate the groups to the new
groups presented in the playbooks.

Security impact
---------------

By ultimately shipping less code, we would marginally
improve our security.

Performance impact
------------------

* Moving from dynamic to static file with the same format doesn't
  change performance
* Moving from static json to static yaml may or may not improve
  performance in your deployment by reducing memory usage.
  It fully depends on the inventory.
  Large inventories are more likely to lose performance
  by switching to yaml for the same input.
* Cleaning up the inventory have a positive performance impact.

End user impact
---------------

The end users will not notice any change.

Deployer impact
---------------

The deployer will have a different user configuration to deal with
(static files)

Hopefully it shouldn't be too hard to understand for an existing
openstack-ansible user, or an experienced ansible user.

Developer impact
----------------

No change for the development of roles or playbooks.

At the same time we are removing technical debt, we are adding new
technical debt by adding these new tools.

With the hope this tools would be easier to understand, read, review,
and having more tests, it would overall reduce risks for the project.

Dependencies
------------

None

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  evrardjp

Other contributors:
  None for now.

Work items
----------

Use static files is not without downsides:
We are losing some key features if we "just use" a
static inventory which is created by the user, like the
dynamic hostname generation, the dynamic IP allocations.

So I propose the following path:

#. We list the groups required for a successful ansible deploy,
   and document those in the reference guide.

   Positive improvements:

   * For deployers that don't want to use our inventory, we
     would now have an "explicit" contract of what they should
     do to run openstack-ansible with their own inventory groups

   Drawbacks:

   * All changes in groups now needs proper documentation
   * That's not enough to come with your own inventory

#. Keep the conf.d/env.d, and dynamic inventory script for now.
   We use it for generating a json that stays static during the
   lifecycle of the cloud, or until re-generated manually. The
   env.d/conf.d/openstack_user_config.yml are used as input
   for this "one-off" run of the dynamic inventory.

   To make sure deployers don't misunderstand the "static" json
   file or confuse it with the current openstack_inventory.json,
   we should move the current files to a "cache" folder, and
   generated the "static" inventory into a ``inventory`` folder.

   Positive improvements:

   * No hidden failures, the generation of the inventory becomes
     a part of the deploy. We can add health checks easily.
   * Our code run only once, during the generation. Therefore we
     are not vulnerable to issues appearing when running
     multiple ansible simulatenously, or other side effects.
   * We keep the container name generation, provider networks,
     and IP assignments for free.

   Drawbacks:

   * Edition of static file will not be in sync with
     conf.d/env.d, but that was already the case with a manual
     change to openstack_inventory.json
   * The inventory_manage script becomes useless

#. We provide default child mapping: we create the x_all groups
   in an easy to read .ini file in the openstack-ansible repo.

   Positive improvements:

   * All our users with their own inventory won't have to
     create EXACTLY the same code to do child group mapping.
     Sharing is caring.
   * We would cary a lot of empty groups, and maybe people don't
     need them.
   * The mapping could then be used to partially replace the
     documentation of step 1, and will fully replace the
     step 1 documentation when the groups will be cleaned
     in the playbooks and roles.

#. We export the host vars into a static files inside the
   userspace inventory folder.

   Positive improvements:

   * Having static yaml files will make it easy to
     see repetitions, and things that can move to
     group vars

   Drawbacks:
   * More static files to maintain by the deployer.
     If we change a host var, we could change the
     inventory and it was applied everywhere.
     It would not be the case anymore.

#. We write a tool manipulating the inventory json.
   By default, that tool would:

   * discard all the groups that aren't listed
     in the reference guide
   * discard all the _all groups from the inventory,
     as they would not be required in the json anymore
     (handled at a previous step)
   * discard all the host variables (handled at a previous step)
   * discard groups that can be generated from facts/host
     variables, like all_containers
     (using group_by would provide the same result).

   Positive improvements:

   * The inventory would be lighter, and therefore require
     less memory to run. It would also run faster and require
     less computing power.

   Drawbacks:

   * All the changes in groups now require a modification of said
     tool, so a good design is necessary to make it easy to change.

#. We document a list of the expected and required
   host/groups variables.

#. We remove all the unnecessary group and host variables
   that were part of the inventory but aren't important anymore
   by using/providing a tool manipulating variable files (yaml),
   or by providing release notes.

#. We document how to export the cleaned up inventory into
   a new YAML file.

#. The generation of conf.d, env.d, and
   openstack_user_config becomes totally optional at
   this point: We know what is required in a build, and
   ask deployers to provide their own group/host mapping.

   At this point it's optional because:

   #. Assignment of hosts into groups can be done by the user
      with a simple .ini/.yaml file + documentation
   #. Standard group structure is provided by default
   #. We have documented the list of host variables, so they
      can be provided by the user
   #. Generating container with their inventory_hostnames
      can be done by the user.
      It's just a series of host variables:
      ansible_host, container_name, container_tech, physical_host.
      It can even be done with a add_hosts and a loop based
      on a new variable like container_names (property of the host).
   #. Assigning and tracking container IPs based on
      cidr_networks, reserved IPs, and already allocated IPs are
      also host variables. Deployers are responsible to
      provide an IP for their containers.
      Example, the lxc_container_create role creates
      IP, network, and interfaces configuration based on
      lxc_container_networks_combined, which a variable taking
      information from the inventory, by combining default
      lxc_container_networks with the "container_networks"
      variable, which is part of the inventory.
      Note: this part can be later replaced by a lookup.
      By using a lookup, we would simplify the inventory,
      by completely removing its container networks of
      the host vars.

#. We provide a script that runs all these actions for the
   user, but also allow step by step editions and manipulations.

#. We provide a new tool to generate a new kind of
   inventory based on what we learned from users, which
   won't necessary use the openstack_user_config, conf.d, or
   env.d. But we have all the time we need to do it better,
   because the expected inventory is not the same as the
   one we did the past.

#. We spin the old inventory out.

Testing
=======

All the work items would be separately tested in the integrated gates.


Documentation impact
====================

Large. The inventory would need a refactor to explain the expectations for
people coming with their inventory, and for people that will use our generation
tool. At the last step, if another tool is provided, it would also require
documenting.

Each step would require modifying the reference, and maybe the operations
guide.

References
==========

None
