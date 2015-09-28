Ceph Block Devices
####################
:date: 2015-07-23 12:00
:tags: storage, ceph

This spec is a proposal to add the ability to configure cinder, glance, and
nova running in an openstack-ansible installation to use an existing Ceph
cluster for the creation of volumes, images, and instances using Ceph block
devices.

* https://blueprints.launchpad.net/openstack-ansible/+spec/ceph-block-devices

Problem description
===================

This implementation should meet the following user requirements:

* Cinder Volume Creation: As a User I want to be able to allocate block
  storage volumes from the Ceph Storage Cluster to individual virtual machines.
* Cinder Boot from Volume: As a User I want to be able to create a virtual
  machine that boots from a block storage device hosted on the Ceph Storage
  Cluster.
* Cinder Snapshots: As a User I want to be able to create a snapshot of one or
  more Cinder Volumes.
* Cinder Backups: As a User I want to be able to use the Ceph Storage Cluster
  as a target for cinder backups.
* Glance Storage: As a User I want to be able to use the Ceph Storage Cluster
  as a backend for glance.
* Nova Ephemeral Storage: As a User I want to be able to allocate nova instance
  storage from the Ceph Storage Cluster.
* Live Migration Support: As an Admin I want to be able to live migrate virtual
  machines that depend upon (i.e. boots from/mounts) a block storage device
  hosted on the Ceph Storage Cluster.  This is inclusive of both Boot from
  Volume and Nova ephemeral storage.

Proposed change
===============

1. Create ceph_client role to handle installation of ceph packages and
   and configuration of ceph.conf file.

2. Update os_cinder role to conditionally allow cinder-volume to create volumes
   in Ceph by setting volume_driver to cinder.volume.drivers.rbd.RBDDriver.

3. Update os_nova role to conditionally allow nova to boot from cinder volumes
   stored in Ceph.

4. Update os_cinder role to conditionally allow cinder-backup to store backups
   in Ceph by setting backup_driver to cinder.backup.drivers.ceph.

5. Update os_glance role to conditionally allow glance to store images in Ceph
   by setting default_store to rbd.

6. Update os_nova role to conditionally allow nova to boot virtual machines
   directly into Ceph by setting libvirt_images_type to rbd.

Alternatives
------------

None

Playbook impact
---------------

See Proposed change above.

Upgrade impact
--------------

No default configurations should be altered with the introduction of these
changes, and therefore there should be no impact to the upgrade of an
existing installation.

Security impact
---------------

OpenStack services require users and keys to interface with Ceph.  This
implementation should encourage the use of separate Ceph users for each
OpenStack service and ensure that configuration files and keys can only be read
by the intended users.

Performance impact
------------------

Enabling this functionality may result in performance increases or decreases
across the OpenStack installation.  This will highly depend on the hardware and
software configuration of the attached Ceph cluster.

End user impact
---------------

Using Ceph block devices may introduce new features visible to the end user,
such as the ability to live migrate an instance from one hypervisor to another.
Additionally, as stated above, there may be visible performance increases or
decreases depending on several different facters.

Deployer impact
---------------

A deployer will need to explicitly update their inventory and set Ansible
variable overrides to a) enable this functionality and b) correctly interface
with an existing Ceph cluster.

Developer impact
----------------

None

Dependencies
------------

None known

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  https://launchpad.net/~mattt416 (mattt)

Other contributors:
  https://launchpad.net/~git-harry (git-harry)
  https://launchpad.net/~david-wilde-rackspace (d34dh0r53)

Work items
----------

1. Create ceph_client role.

2. Update os_cinder role to conditionally allow cinder-volume to create volumes
   in Ceph.

3. Update os_nova role to conditionally allow nova to attach cinder volumes
   stored in Ceph.

4. Update os_cinder role to conditionally allow cinder-backup to store backups
   in Ceph.

5. Update os_glance role to conditionally allow glance to store images in Ceph.

6. Update os_nova role to conditionally allow nova to boot virtual machines
   directly into Ceph.

Testing
=======

No gate-related adjustments will be made to openstack-ansible to support
this change as no default configurations are being changed here.  Additionally,
that there are strict limitations on what can run in the all-in-one (AIO) gate
instance.

Documentation impact
====================

Documentation will need updating to include:

1. How to enable Ceph block devices for each cinder, glance, and nova services
   and what each newly introduced Ansible variable does.
2. What additional steps are required to be executed on the existing Ceph
   cluster to allow the OpenStack installation to interface with the Ceph
   cluster.

References
==========

None
