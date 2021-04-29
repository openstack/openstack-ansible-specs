Protecting OpenStack Plaintext Secrets Automation
##################################################
:date: 2021-04-22 22:00
:tags: protecting plaintext configs, oslo.config, castellan, vault, security

OpenStack services require sensitive data to be in configuration
files. These are values for various configuration options such as
``password``, ``transport_url``, ``connection``, and so on. The configuration
files for OpenStack services are just plaintext files.
Because of this, even with proper file permissions set on these files,
secret data is kept there without any protection at present.

A specification on protecting plaintext secrets in OpenStack was created
by **Raildo Mascena de Sousa Filho** [1]. Also, the osloconfig and castellan
libraries support the following scheme of configuration files processing:
oslo-config can read configuration options and their values with
help of castellan, which is able to read that data from a
protected key management solution, such as HashiCorp Vault.

The proof of concept for this scheme was made by **Moisés Guimarães
de Medeiros** in his code [2]. So, the problem of securing sensitive options
in OpenStack plaintext configuration is almost solved, with the exception
of the automation of such a secure configuration.

Problem description
===================

At present, ``oslo.config`` allows us to specify the ``config_source`` in
the ``DEFAULT`` section of a configuration file, and to use ``castellan``
driver to read configuration options from the appropriate
configuration file section:

.. code:: bash

 [DEFAULT]
 config_source = secrets

 [secrets]
 driver=castellan
 config_file=castellan.conf
 mapping_file=mapping.conf

The ``castellan.conf`` and ``mapping.conf`` configuration files includes
information on how to read configuration options values from a
secret store.

``castellan.conf`` example:

.. code:: bash

 [key_manager]
 backend=vault

 [vault]
 kv_mountpoint=kv
 vault_url='https://vault.enterprise.local:8200'
 use_ssl=True

``mapping.conf`` example:

.. code:: bash

 [oslo_messaging_notifications]
 transport_url=6d1c6b6bd925418eb3c99523750bc4be

 [database]
 connection=20921387a863462dae4db253198156ec

... where the values of ``transport_url`` and ``connection`` parameters
are IDs of appropriate records in the HashiCorp Vault.

The problem is that a system administrator needs to insert
appropriate values to the HashiCorp Vault or another secret
storage, and then reconfigure the OpenStack services either
manually or with some preferred automation tools. There is no
existing solution today for starting OpenStack with secured
configuration files from the very beginning.

With all of this in mind, we are proposing that system
administrators should be allowed to install OpenStack with
secured configuration files.

Proposed change
===============

So, there are two parts of protecting OpenStack plaintext
configuration files:

* installation of HashiCorp Vault, in case it is not installed;

* proper configuration of all OpenStack services with plaintext configs.

This spec proposes to make the following additions to
``openstack-ansible`` [3]:

* add playbook for HashiCorp Vault installation;

* add additional tasks to OpenStack services playbooks;

* add additional parameters for system administrator to choose if OpenStack
  is going to be installed with protected plaintext configs or not.

Such additional parameters are:

* ``vault_hosts`` – optional parameter which indicates hosts where HasiCorp
  Vault should be installed. HashiCorp Vault should have high availability
  installation as other OpenStack services have;

* ``protected_configs`` and ``protected_configs_castellan_conf``, should be added
  to ``openstack_user_config.yml`` file.  ``protected_configs`` parameter should
  take ``true`` or ``false`` values, and the default value should be ``false``.

If the system administrator sets the value to ``true``, then
additional steps are performed in playbooks to address the
below:

* check for required Python libraries (such as ``castellan``) and install i
  them if needed;

* add appropriate values to secret store;

* prepare service configuration file without sensitive data
  but with ``config_source`` option and secrets section;

* add ``castellan.conf`` to service configuration directory;

* add ``mapping.conf`` to service configuration directory.

This change does not require any changes to ``oslo.config`` or
``castellan``, since everything is already supported at present.

Alternatives
============

The protection of plaintext secrets can be implemented by
using a separate Ansible playbook just for the services
reconfiguration, after installing OpenStack with unsecured
configuration files with ``opentack-ansible``.

In that case, the list of services to secure should be given to the
playbook, and the tasks of such a playbook should do almost the
same steps as described above:

* install and initialize HashiCorp Vault;

* check for required Python libraries (such as ``castellan``)
  and install them if needed;

* add appropriate values to secret store;

* prepare service configuration file without sensitive data
  but with ``config_source`` option and secrets section;

* add ``castellan.conf`` to service configuration directory;

* add ``mapping.conf`` to service configuration directory;

* restart OpenStack service.


Playbook/Role impact
--------------------

Additional role on HashiCorp Vault installation should be added.

Additional tasks for proper configuration of OpenStack services should
be added to playbooks.


Upgrade impact
--------------

No impact.

Security impact
---------------

Sensitive data is going to be removed from plaintext config files
so security will be improved with this change.

Performance impact
------------------

No impact.

End user impact
---------------

No impact.

Deployer impact
---------------

No impact in case deployer does not want to protect plaintext configs.
Otherwise deployer will be able to configure additional options to
remove sensitive data from plaintext configs as described above.


Developer impact
----------------

No impact.

Dependencies
------------

Here is initial blueprint regarding protection of sensitive data inside
plaintext configs:

https://blueprints.launchpad.net/oslo.config/+spec/protect-plaintext-passwords

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  yeremko
  <Alexander.Yeremko@walmart.com>

Other contributors:
  <Misha.Vorona@walmart.com>

Work items
----------

* Implement changes to ``openstack-ansible`` [3]

* Implement changes to ``openstack-ansible-tests`` [4]

* Update documentation.

Testing
=======

Additional tests in ``openstack-ansible-tests`` [4] will be required to
cover the added functionality.

Documentation impact
====================

The documentation will need to be updated to illustrate how to
protect plaintext configs and work with them further.

References
==========

[1] https://specs.openstack.org/openstack/oslo-specs/specs/stein/secret-management-store.html

[2] https://github.com/moisesguimaraes/ep19

[3] https://opendev.org/openstack/openstack-ansible

[4] https://opendev.org/openstack/openstack-ansible-tests

[5] https://www.vaultproject.io/


