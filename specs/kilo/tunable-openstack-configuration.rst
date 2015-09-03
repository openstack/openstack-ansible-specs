Tunable OpenStack Configuration
###############################
:date: 2015-08-26
:tags: openstack, configuration, tuning

Instead of implementing a specific variable for each possible/desired
configuration entry in every role, there is a more general way that this
could be done which would enable the deployer to implement any desired
valid configuration entry for a given role. OpenStack Ansible will then
only need to implement deviations from the standard OpenStack default
settings. Examples of these settings would be minimum settings to make
the service work and best practice settings. Changes to templates would
allow the project the ability to ship flat, or otherwise minimally dynamic,
configuration files in an effort to limit the sheer size and scope of the
number of variables we have if a given role.

* https://blueprints.launchpad.net/openstack-ansible/+spec/
  tunable-openstack-configuration

This implementation is intended to:

* Reduce the cruft of variables the project needs to carry just to enable
  customizations requested from deployers.

* Reduce the amount of configuration file settings the project needs to work
  through for every major upgrade of the underlying OpenStack environment.

* Decrease the turnaround time for deployers to implement additional
  configuration items in OpenStack configuration files.


Problem description
===================

The OpenStack Ansible project currently carries a lot of variables in
roles which are simply there to enable the ability to override OpenStack
default settings. This is unnecessary cruft which has to be reviewed for
deprecated options whenever the project starts working with an updated
version of OpenStack.

It also introduces an unnecessary propose/develop/test/release cycle for
simple changes to configuration files which could easily be done by a
deployer if they're enabled to do so.

Proposed change
===============

* A new Ansible *action plugin* will be created which will facilitate the ability
  for templates to be updated dynamically. This change will build off of the
  existing Ansible template functionality and but allow for changes to be applied
  through a simple dictionary data structure. Updates are applied to the
  template while in transit allowing us to carry minimal code while leveraging all
  of the already built in Ansible functionality.

* The action plugin named "config_template" will add two new input types
  `config_data` and `config_type`. The new input options will be optional allowing
  us to simply replace our current template tasks with the `config_template` module.

* New defaults will be created as empty dictionaries as base entry points for
  deployers to override items in config.


Code wise the change to a templed task will look something this:

.. code-block:: yaml

  - name: run config template ini
    config_template:
      src: templates/test.ini.j2
      dest: /tmp/test.ini
      config_data: {'data': 'things'}
      config_type: ini

  - name: run config template json
    config_template:
      src: templates/test.json.j2
      dest: /tmp/test.json
      config_data: {'data': 'things'}
      config_type: json

  - name: run config template yaml
    config_template:
      src: templates/test.yaml.j2
      dest: /tmp/test.yaml
      config_data: {'data': 'things'}
      config_type: yaml


To note:
  A dictionary used to update or override items within a configuration template.
  The dictionary data structure may be nested. If the target config file is an ini
  file the nested keys in the ``config_data`` will be used as section headers.

Alternatives
------------

Continue to use the current paradigm of adding an ansible variable per
configuration configurable file entry.

Playbook impact
---------------

The existing roles would need to be adjusted to support the new config_data entry
point and to have the relevant template tasks updated to the config_template module.

Upgrade impact
--------------

This change will not impact current upgrades as everything in all of the templates
can remain the same. The proposed module changes simply make it possible for
deployers to update templates as needed without having to make changes in tree. In
future releases we can deprecate variables we're presently carrying as needed or
wanted while still maintaining the ability to override options as needed through the
use of the config_template module.

Security impact
---------------

None.

Performance impact
------------------

None.

End user impact
---------------

None

Deployer impact
---------------

Deployers will be able to dynamically update configuration options based on their
need without making changes in tree. This will allow deployers a greater ability
to tailor deployments as needed.

Developer impact
----------------

This would reduce the need for developers to get involved with small patches
that implement basic configuration file entries which deployers wish to use.

Dependencies
------------

None.

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  https://launchpad.net/~kevin-carter
  IRC: cloudnull

Other contributors:
  None at this time.

Work items
----------

Implement tunable configurations for all configuration files that fall under
the following formats: [yaml, json, ini]

* Develop Ansible Action Plugin to enable the ability to make in flight config
  changes to an existing template.

* Change all template tasks within the roles that drop configuration files to
  use the new config_template module.

* Replace the `copy_update` module with the `config_template` module.


Testing
=======

In the current gate testing we can add a basic template test to override a few
options / add a few options and assert that the changes from the base template
took place. This can be accomplished using items from the example tasks and a
simple json, ini, and yaml data structure. We could also set overrides with the
gate that we know we want to run within our deployments such that we're
exercising the OpenStack code paths that we're attempting to enable via the
gate. In this way we might be able to cut out some of our gate script variables
as well.


Documentation impact
====================

While the Action Plugin has documentation within it, per the normal Ansible
module documentation process, we can also update our general install
documentation to reference the existence of the new module and how it works.
I'd like to refrain from documenting every override entry point as the
authoritative source for those types of items will be the role "defaults"
themselves.

References
==========

None
