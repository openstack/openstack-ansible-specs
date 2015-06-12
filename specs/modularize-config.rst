Modularize configuration files
##############################
:date: 2015-05-08 00:00
:tags: config, configuration, modularize, modular

Modularize deployment configuration files to simplify the configuration
process.

Blueprint:

https://blueprints.launchpad.net/openstack-ansible/+spec/modularize-config

Problem description
===================

Deployment configuration primarily occurs in a rather monolithic
``openstack_user_config.yml`` file. Although adding documentation
to this file eases understanding of the various levels and options,
it also increases the size and apparant complexity, especially for
larger deployments. With the addition of swift, the configuration
structure already supports configuration files in the conf.d directory.
Splitting the main monolithic configuration file into smaller files
containing similar components helps overall organization, especially
for larger deployments.


Proposed change
===============

Similar to swift, modularize similar sections of configuration files,
particularly ``openstack_user_config.yml``, into the following separate
files in the conf.d directory.

* hosts.yml

  * Includes configuration for target hosts with simple options. For
    example, ``repo_hosts`` typically contains only a list of hosts.
    In comparison, ``storage_hosts`` requires significantly more options
    and should therefore use a separate file.
  * Contains the following levels:

    * ``shared-infra_hosts``
    * ``repo_hosts``
    * ``os-infra_hosts``
    * ``identity_hosts``
    * ``network_hosts``
    * ``compute_hosts``
    * ``storage-infra_hosts``
    * ``swift_proxy-hosts``
    * ``log_hosts``

  .. note::
     For consistency, consider changing ``swift_proxy-hosts`` to
     ``swift-proxy_hosts`` and ``swift_hosts`` to ``swift-storage_hosts``.

* networking.yml

  * Includes host networks, IP address blacklist for inventory generator,
    load balancer options, and provider networks.
  * Contains the following levels:

    * ``cidr_networks``
    * ``used_ips``
    * ``provider_networks`` (from ``global_overrides``)
    * ``internal_lb_vip_address``, ``external_lb_vip_address``,
      ``management_bridge``, and ``tunnel_bridge`` (from ``global_overrides``)

* cinder_storage_hosts.yml

  * Includes configuration for cinder storage target hosts with complex
    options for backends.
  * Contains the following level:
    * ``storage_hosts``

* swift_storage_hosts.yml

  * Includes configuration for swift storage target hosts with complex
    options.
  * Contains the following levels:
    * ``swift`` (from ``global_overrides``)
    * ``swift_hosts``


Alternatives
------------

Use a different strategy to modularize the configuration files or keep the
existing monolithic structure.


Playbook impact
---------------

None.


Upgrade impact
--------------

Optionally, modularize configuration files according to this specification
before or after upgrading to a version that supports it.


Security impact
---------------

None.


Performance impact
------------------

None.


End user impact
---------------

None.


Deployer impact
---------------

Simplify the configuration process.


Developer impact
----------------

Developers should consider the modular configuration when adding or changing
configuration items.


Dependencies
------------

None.


Implementation
==============

Assignee(s)
-----------

Primary assignee:
  <ionosphere80> **Sam-I-Am**


Work items
----------

* As necessary, break existing monolithic configuration files into smaller
  files that contain groups of similar items and reside in a ``.d``
  directory within the configuration file structure.


Testing
=======

* Verify changes do not break gating process. The AIO script for gating can
  continue to use a monolithic file or modular files with the ``.aio``
  extension.


Documentation impact
====================

* Change documentation that references monolithic configuration files to
  reference modular configuration files.


References
==========

None.
