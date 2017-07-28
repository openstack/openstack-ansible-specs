Use dnf with CentOS
###################
:date: 2017-07-28 00:00
:tags: centos, dnf, packaging

Blueprint: `Use dnf with CentOS`_

.. _Use dnf with CentOS: https://blueprints.launchpad.net/openstack-ansible/+spec/centos-and-dnf

CentOS 7 currently uses ``yum`` as its default package manager.  However,
Fedora has moved to ``dnf`` for several releases and it provides significant
performance benefits. It can make the metadata cache, evaluate dependencies,
and handle fastest mirror checks much more efficiently.

The ``dnf`` and ``yum`` package managers can co-exist together without causing
conflicts.  Several Fedora releases ran both of these simultaneously. The
``dnf`` packages are available in the EPEL repositories (which we currently
enable). It uses all of the existing ``yum`` repositories and GPG keys as well.

Problem description
===================

The CentOS gate jobs are notoriously slow and the integrated gate times out on
tempest runs frequently. The longest running tasks in each role involve the
installation of distro packages because these tasks use ``state: latest`` the
``yum`` tasks.

When Ansible sees ``state: latest``, it goes through a fairly tedious process:

* Run ``check-update``, which checks the **entire** system for updates.
* If some packages are returned (they need updates), Ansible searches the list
  to see if any packages from the ``yum`` task are in that list.
* If some packages need updates, Ansible calls ``yum`` to install those
  packages.

This process can take 5-8 seconds even for *one* package. In comparison,
``dnf`` completes the task in 0.8-1.6 seconds. This should give us some wiggle
room to get CI jobs completed sooner and convert more of the CentOS jobs from
non-voting to voting.

Proposed change
===============

On CentOS systems, we should install ``dnf`` and ``python-dnf`` (for Ansible
compatibility). Ansible will prefer ``dnf`` over ``yum``, so we would need to
ensure that each role has support for ``dnf`` tasks.  Since both package
managers are interchangeable, this could be done by symlinking the
``*_install_dnf.yml`` task files to ``*_install_yum.yml`` and using the
``package`` module in those task files.

Alternatives
------------

If ``dnf`` isn't preferred, we could avoid using ``state: latest`` for CentOS
installations.  This would cause CentOS deployments to diverge from Ubuntu
and OpenSUSE deployments and it would make bug triage more challenging.

Another option is to update the entire system when ``state: latest`` is
provided but switch all of the package installation tasks to use ``state:
present``. This will save us a small amount of time since Ansible will skip the
``check-update`` step and go straight into updating all packages. This would
be another diversion from the Ubuntu/OpenSUSE process, however.

Playbook/Role impact
--------------------

Each role with a set of ``yum`` tasks would need to be converted to use
``package``. A symlink would be needed so that CentOS systems with ``dnf``
installed would use the same tasks.

Upgrade impact
--------------

During the upgrade process, ``dnf`` would be installed on CentOS systems.
Ansible would begin to use ``dnf``, but the deployer could continue using
``yum`` for their own administration tasks if they prefer it.

Security impact
---------------

The ``dnf`` package manager supports the same configuration options as yum for
checking GPG keys of packages and repositories.

Performance impact
------------------

The ``dnf`` package manager will provide better performance when managing
packages, but the rest of the system will perform at the same levels.

End user impact
---------------

End users will not notice this change or gain any benefits from it.

Deployer impact
---------------

Deployers may notice that some roles use ``dnf`` while others use ``yum`` until
all of the patches have merged. This won't affect the running system, but it
may make some playbooks faster than others.

Deployers would continue to deploy in the same ways that they currently do
today.

Developer impact
----------------

Developers must be aware that ``dnf`` is present on CentOS systems and that
Ansible will prefer it over ``yum``.  Any new roles/playbooks or updates to
existing ones will need to include support for ``dnf`` via the ``dnf`` module
or the ``package`` module (which selects ``dnf`` over ``yum`` already).

Dependencies
------------

This spec is not dependent on any other spec or blueprint.

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  Major Hayden (IRC: mhayden, Launchpad: rackerhacker)

Work items
----------

* Add ``dnf`` patches to the base roles first (openstack_hosts, lxc_hosts, etc)
* Continue moving up the dependent roles until all roles include
  ``dnf``-compatible tasks
* Ensure that the integrated repository and openstack-ansible-tasks use ``dnf``

Testing
=======

The existing testing done in the OpenStack CI jobs will be sufficient for this
work. If ``dnf`` is not installing packages properly or efficiently, we will
see that reflected in the testing playbooks.

Documentation impact
====================

This work will require some release notes to notify developers and deployers of
the ``dnf`` change. However, there's no need for extensive documentation since
``dnf`` supports the same configurations and arguments as ``yum``.

References
==========

* Test patch for openstack-ansible-openstack_hosts:
  https://review.openstack.org/488268

* Vultr docs for dnf on CentOS 7:
  https://www.vultr.com/docs/use-dnf-to-manage-software-packages-on-centos-7
