Pluggable Inventory Backends
############################
:date: 2016-12-13 22:00
:tags: inventory, craton

This spec is intended to provide guidance and a longer term goal for migrating
the existing inventory system from a single, coupled filesystem interface to
a pluggable system supporting multiple backends for storage.

  * https://blueprints.launchpad.net/openstack-ansible/+spec/inventory-pluggable-backends

Currently, the generated inventory is kept in one place - the
``/etc/openstack_deploy/openstack_inventory.json`` file on the deployment
node. While this has worked, it is not very robust. In order to accommodate
more deployer flexibility, the inventory system should be reworked to use a
pluggable system for storing necessary info. A filesystem plugin would provide
backwards compatibility, and a Craton plugin will also be developed.


Problem description
===================

While there are multiple issues to be addressed in the current codebase,
this spec focuses solely on the storage of inventory facts.

Since OpenStack-Ansible's creation in Icehouse, the source of truth for
a completed inventory has been the
``/etc/openstack_deploy/openstack_inventory.json`` file. While this has
worked, it has a few drawbacks:

* As a single file, it may be deleted by accident. If the configuration files
  have changed, getting an exact copy back without the tar backup files is
  impossible due to UUID suffixes.

* There are only UNIX file permissions managing access to the file.

* No accounting of changes, besides a simple tar backup, exists. This is
  useful for documentation and auditing of a running cluster.

This spec does not aim to define a fully robust inventory management
system. Instead, the OpenStack-Ansible inventory system can be made more
modular, especially in its storage, so that deployers can take advantage of
other, dedicated systems.

Doing so requires refactoring the dynamic inventory generation code so that
storage concerns, such as writing the output and reading output from previous
runs, are no longer directly tied to the generation of values.

Proposed change
===============

Any code that interfaces with the filesystem currently will be moved into its
own Python module, so that it exists separate from the generating logic. This
work is already happening, and doesn't rely on a particular plugin system in
order to be completed.

Once that is done, a plugin system will be used to provide
a generic interface for storage actions to the rest of the codebase. This code
will connect to the plugin library, and return a compatible instance of a storage
plugin.

Plugin Python API
-----------------

Plugins should support the following public methods:


register
  registers a plugin with the plugin system. Receives a dictionary
  of all arguments specified for plugins, and should extract any information
  it needs, such as file locations, connection strings, or URLs.


load
  loads data from source into a Python dictionary in the current dynamic
  inventory scripts. Source may be a file, a database, or any other backend
  system.


write
  Writes the inventory dictionary to the specified backend. Receives
  the inventory dictionary as an argument


Configuration Changes
---------------------
Some new configuration will likely need to be presented to the user, so that
the appropriate plugins for storage can be identified. This configuration is
not well suited to the ``etc/openstack_deploy/openstack_user_config_file``,
since part of the script's job entails accessing that file.

Instead, the ``/etc/ansible/osa.ini`` file is proposed, to match the
style ec2.py script's method laid out in `Ansible's dynamic
inventory documentation`_.

The specific format of this file will be left to implementation reviews,
however at a high level it will likely include a Python import path to the
inventory module to use, and any settings it may need such as connection
strings or API URLs.

Alternatives
------------

Nothing could be done and the inventory could continue to rely on the JSON
backend; it's worked reliably until now. However, its inherent problems would
remain.

In the current state, deployers must fork the repository to modify the dynamic
inventory code, which is a maintenance burden long term, likely unsustainable.

Alternate inventory scripts could also be placed in ``playbooks/inventory``,
either replacing or adding to the current ``dynamic_inventory.py`` file.
Replacements may or may not use the current ``openstack_user_config.yml`` file
and environment structure. Additional scripts would produce output merged from
all sources, read using ``os.listdir`` and then processed according to
alphanumerical file name.(see the `Ansible script loader`_).

Configuration Alternatives
**************************

Rather than the ``/etc/ansible/osa.ini`` file, environment variables could
be used to set plugin options.

Plugin Implementation Alternatives
**********************************
Existing plugin implementations include `PluginBase`_ and `Stevedore`_.

Stevedore is an OpenStack project, and using it would align the project more
closely with the community. Stevedore requires registering plugins
via `setup.py` `entry_points`. The ``setup.cfg`` for OpenStack-Ansible needs
modifications to install the python code, and the ``lib`` directory needs
to be renamed in order to work within pbr's design. A `prototype review
<https://review.openstack.org/#/c/418076/>` for these changes has been
submitted.

PluginBase is not directly part of the OpenStack ecosystem, but is simple,
standalone plugin system. It has no external dependencies, and can be used via
the standard Python import system without requiring a `setup.py` file for our
existing code.

The PluginBase method is less impactful for in-tree code, however any
external plugins should be pip-installable anyway, thus having a `setup.py`.
Therefore, either is viable as an implementation option, with Stevedore
requiring slightly more upfront work.

Playbook/Role impact
--------------------

Ideally, playbook impact should be minimal or non-existent. The inventory
generated should look the same from a playbook perspective, regardless of
backend plugins used.

Upgrade impact
--------------

A migration path from the existing JSON source should be provided, so that
existing environments can move to new systems, should they choose. However,
migrating will likely involve implementation detail knowledge that differs per
system, so each one will likely need to have its own import functionality.

An `export function`_ has already been implemented to provide the inventory in
a per-host format. This could be used as a basis for external systems to use
in their own import systems.

This export/import process is assumed to be out-of-band from the playbook
runs.


Security impact
---------------

This change introduces communication to outside systems - there is inherent
risk in doing so. These systems are assumed to be using secure channels and
trusted by the deployers.

Secrets could theoretically be stored in these backends, though the
``openstack-ansible.sh`` wrapper script currently references the
``user_secrets.yml`` file instead of placing those in the inventory. The
system should not dictate that this is the only solution for secrets, however.
Where deployers choose to put these is up to them, though storing secrets in
any sort of unencrypted or unprotected backend is not advised.


Performance impact
------------------

Inventory may take longer to generate or look up depending on the system used
as a backend. If said system is used via a network interface, latency and
caching are concerns.

Since this category is fairly broad and different systems will have different
characteristics, more detail is best left to specific plugin implementations.

End user impact
---------------

Users of the deployed clouds, those consuming virtual machines and networks,
should not see much of a difference. This change is largely to facilitate
deployer concerns.


Deployer impact
---------------

Deployers may have entirely new inventory backend sources. Configuration
options for reading from said sources would have to be provided. The default,
in-tree implementation is likely to remain the JSON file for the time being.

Changes to existing clusters will require deployer intervention to migrate
relevant data from the file into their new system, which may or may not be
managed externally to an OpenStack-Ansible deployment.

Any helper scripts that relied on the ``openstack_inventory.json`` file will
need to be modified, preferably to take advantage of the new plugins/APIs.

The ``inventory-manage.py`` script currently only provides a management
interface for the JSON file, and is not intended to be a universal
inventory management tool. Different systems will have their own clients
or front ends for doing such management and querying.


Developer impact
----------------

Developers of roles should be able to rely on the inventory information
staying the same.

Developers working on the inventory generation must account for multiple
backend sources of data, however the intention is to provide a uniform API
for working with that data.


Dependencies
------------

While not a strict dependency, this is closely related to the
`dynamic inventory lib`_ blueprint/spec. It specifically tries to solve
the problem of external backends.

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  nolan-brubaker (IRC: palendae)

Other contributors:
  steve-lewis (IRC: stevelle)

Work items
----------

* Implement existing code as a separate module. This work has largely been
  done, see code reviews `https://review.openstack.org/#/c/392056/`,
  `https://review.openstack.org/392417`, and `https://review.openstack.org/399303`.

* Implement plugin system. Whether this is Stevedore or PluginBase, the code
  for interfacing with the system will need to be written.

* Implement the file system code as a plugin. This may be done in tandem with
  the previous item in order to fully test it.


Testing
=======

Unit and integration tests should be written to ensure that the existing JSON
code continues to work. Also, sample plugins should be written to exercise the
system, even if they are 'dummy' systems.

Since this fits into inventory tests, it should not affect the integrated gate
build times. It should also be tested per-commit.

Testing will also be implicit in the integrated build, but not necessarily
targeted for easy troubleshooting.

Individual plugins external to this repo will need to be gated separately.


Documentation impact
====================

Guidance on writing plugins and migrating to new systems should be provided

References
==========

This spec has been informed by discussions on etherpads such as:

* `https://etherpad.openstack.org/p/osa-dynamicinventory-plugins`
* `https://etherpad.openstack.org/p/craton_osa`
* `https://etherpad.openstack.org/p/openstack-ansible-newton-dynamic-inventory`

.. _`Ansible's dynamic inventory documentation`: http://docs.ansible.com/ansible/intro_dynamic_inventory.html#example-aws-ec2-external-inventory-script
.. _`PluginBase`: http://pluginbase.pocoo.org/
.. _`Stevedore`: http://docs.openstack.org/developer/stevedore/tutorial/index.html
.. _`export function`: https://review.openstack.org/#/c/371798/
.. _`dynamic inventory lib`: https://blueprints.launchpad.net/openstack-ansible/+spec/dynamic-inventory-lib
.. _`Ansible script loader`: https://github.com/ansible/ansible/blob/392232895491aca2e4f65331131103f4f2ea5b7d/lib/ansible/inventory/dir.py#L95-96
