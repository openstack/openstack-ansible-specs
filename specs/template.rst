Example Spec - Title of your spec
#################################
:date: 2015-02-02 22:00
:tags: used, for, groupings, and, indexing

Update the date of your spec with the date that it was proposed.
Add any tags to the spec that may be of use for people to get what its
about at a glance.

Provide a synopsis as to why you are creating this spec/blueprint.

Include the URL of your launchpad blueprint:
  * https://blueprints.launchpad.net/openstack-ansible/+spec/example

Introduction paragraph -- why are we doing anything? A single paragraph of
prose that operators can understand. Describe the problem in as much detail
as you can.

Some notes about using this template:

* Your spec should be in ReSTructured text, like this template.

* Please wrap text at 79 columns.

* The filename in the git repository should match the launchpad URL, for
  example a URL of: https://blueprints.launchpad.net/openstack-ansible/+spec/awesome-thing
  should be named awesome-thing.rst

* Please do not delete any of the sections in this template.  If you have
  nothing to say for a whole section, just write: None or N/A

* For help with syntax, see http://sphinx-doc.org/rest.html

* To test out your formatting, build the docs using tox, or see:
  http://rst.ninjs.org

* If you would like to provide a diagram with your spec, ascii diagrams are
  required.  http://asciiflow.com/ is a very nice tool to assist with making
  ascii diagrams.  The reason for this is that the tool used to review specs is
  based purely on plain text.  Plain text will allow review to proceed without
  having to look at additional files which can not be viewed in gerrit.  It
  will also allow inline feedback on the diagram itself.


Problem description
===================

A detailed description of the problem:

* For a new feature this might be use cases. Ensure you are clear about the
  actors in each use case: End User vs Deployer.

* For a major reworking of something existing it would describe the
  problems in that feature that are being addressed.


Proposed change
===============

Provide an overview of the changes you'd like to see implemented. Here is
where you cover the change you propose to make in detail. How do you propose
to solve this problem?

Notable changes:
  * list out all of the notable changes.

If this is one part of a larger effort make it clear where this piece ends. In
other words, what's the scope of this effort?


Alternatives
------------

What, if any, are the alternatives to the changes you are proposing? What other
ways could we do this thing? Why aren't we using those? This doesn't have to be
a full literature review, but it should demonstrate that thought has been put
into why the proposed solution is an appropriate one.


Playbook impact
---------------

What impact will there be on the playbooks?


Upgrade impact
--------------

If this change set concerns any kind of upgrade process, describe how it is
supposed to deal with that stuff. For example, if containers are removed and
or have their specific purpose changed how do you indend to deal with the
eventual upgrade from the prospective of an existing installation? Does this
change require documentation to be fully supported or will there be specific
tooling that has to be created in order for the upgrade to be completed?


Security impact
---------------

Describe any potential security impact on the system.  Some of the items to
consider include:

* Does this change touch sensitive data such as tokens, keys, or user data?

* Does this change alter a deployed OpenStack API in a way that may impact
  security, such as a new way to access sensitive information or a new way to
  login?

* Does this change involve cryptography or hashing?

* Does this change require the use of sudo or any elevated privileges?

* Does this change involve using or parsing user-provided data? This could
  be directly at the API level or indirectly such as changes to a cache layer.

For more detailed guidance, please see the OpenStack Security Guidelines as
a reference (https://wiki.openstack.org/wiki/Security/Guidelines).  These
guidelines are a work in progress and are designed to help you identify
security best practices.  For further information, feel free to reach out
to the OpenStack Security Group at openstack-security@lists.openstack.org.


Performance impact
------------------

Describe any potential performance impact on the system. For example, how is
the code executed and does it depend on upstream resources that may be
unavailable?

Examples of things to consider here include:

* Adding a new PPA for upgraded packages, who is the maintainer? Does this
  individual push often? How long has this individual/company maintain
  specific packages?

* Adding additional pinned packages for use in a python wheel. Does this
  package change often? Are there tests?


End user impact
---------------

How would the end user be impacted by this change? The "End User" is defined
as the users of the deployed cloud?


Deployer impact
---------------

How would the deployer be impacted by this change? Discuss things that
will affect how OpenStack will be deployed, such as:

* What config options are being added? Should they be more generic than
  proposed? Are the default values ones which will work well in
  real deployments?

* Is this a change that takes immediate effect after its merged, or is it
  something that has to be explicitly enabled?

* If this change is a new binary, how would it be deployed?

* Please state anything that those doing continuous deployment, or those
  upgrading from the previous release, need to be aware of. Also describe
  any plans to deprecate configuration values or features.  For example, if we
  change the name of a play, how do we handle deployments before the change
  landed?  Do we have a special case in the code? Do we assume that the
  operator will recreate containers within the infrastructure of the cloud?
  Does this effect running instances within the cloud?


Developer impact
----------------

How does this change impact future developers working on the ansible
playbooks? Discuss things that will affect other developers working on
OS-Ansible-Deployment, such as:

* If this spec proposes a new role, how will that role be deployed? Is this a
  new default role? Does this role have a host impact?


Dependencies
------------

Does this blueprint/spec depend one another blueprint or spec?

* Include specific references to specs and/or blueprints in
  os-ansible-deployment, or in other projects, that this one either depends on
  or is related to.

* Is the new requirement due to an upstream change? If so document it and
  provide references to the change.


Implementation
==============

Assignee(s)
-----------

Who is leading the writing of the code? Or is this a blueprint where you're
throwing it out there to see who picks it up?

If more than one person is working on the implementation, please designate the
primary author and contact.

Primary assignee:
  <launchpad-id or None>

Other contributors:
  <launchpad-id or None>

Please add **IRC nicknames** where applicable.

Work items
----------

Work items or tasks -- break the feature up into the things that need to be
done to implement it. Those parts might end up being done by different people,
but we're mostly trying to understand the timeline for implementation.


Testing
=======

Please discuss how the change will be tested. You should be able to answer the
following questions:

* Does this change impact how gating is done?

* Can this change be tested on a **per-commit** basis?

* Given the instance size restrictions, as found in OpenStack Infra
  (8GB Ram, vCPUs <= 8), can the test be run in a resource constrained
  environment?

* Is this untestable given current limitations (specific hardware /
  software configurations available)? If so, are there mitigation plans
  for this change to be tested within 3rd party testing, gate enhancements,
  etc...?

* If the service is not OpenStack specific how can we test the change?


Documentation impact
====================

What is the impact on the docs team of this change? Some changes might require
donating resources to the docs team to have the documentation updated. Don't
repeat details discussed above, but please reference them here.


References
==========

Please add any useful references here. You are not required to have any
reference. Moreover, this specification should still make sense when your
references are unavailable. Examples of what you could include are:

* Links to mailing list or IRC discussions

* Links to relevant research, if appropriate

* Related specifications as appropriate

* Anything else you feel it is worthwhile to refer to

