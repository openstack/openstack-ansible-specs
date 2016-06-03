Support Xen Virt Driver
#######################
:date: 2016-06-03 11:17
:tags: ansible, xen

The purpose of this spec is to add support for the Xen Hypervisor to
OpenStack-Ansible. This will allow the use of Xen as an option on OpenStack
compute nodes.

https://blueprints.launchpad.net/openstack-ansible/+spec/xen-virt-driver


Problem description
===================

Xen is a tested and supported hypervisor in OpenStack.  It is used in some of
the largest public clouds today and would make a good addition to
OpenStack-Ansible.  Support for Xen exists in the OpenStack Libvirt Driver today
so implementation should not be difficult.

Proposed change
===============

The primary change is to add support in OpenStack-Ansible for Xen on CentOS 7,
Ubuntu 16.04, and Ubuntu 14.04 (by using UCA repos).  The necessary
`changes <http://wiki.xenproject.org/wiki/OpenStack_CI_Loop_for_Xen-Libvirt>`_
for Xen to work with OpenStack are in Xen 4.5.1 and Libvirt 1.2.15.  This
blueprint covers deploying the nova-xen compute driver with the standard
networking agents that OpenStack-Ansible supports.

The proposed changes include:

* Add support for installing/configuring the Xen virt driver and dependencies
* Documentation for how to configure a compute to run the Xen virt driver
* Tests to verify changes to the os_nova role required for Xen support


Alternatives
------------

* Maintain independent Xen Ansible playbooks - This requires reinvention
  of base function and does not meet operator requirements.


Playbook/Role impact
--------------------

See the Work Items for the playback/role impact.  References to nova_virt_type
will be updated to reflect a 'xen' option.


Upgrade impact
--------------

None. The xen driver is new for OpenStack-Ansible, and as such has no upgrade
impact.


Security impact
---------------

None.


Performance impact
------------------

None.


End user impact
---------------

End users will be able to deploy compute nodes using the Xen virt driver.


Deployer impact
---------------

Xen specific configuration options will be added to the
openstack-ansible-os_nova role.

When support for Xen as a virt driver is added these config options will be
available for use; however it is expected that Xen support will be disabled by
default, requiring that deployers explicitly enable Xen support and configure
hosts for OpenStack-Ansible to use.

Documentation of these new configuration items will be provided and a set of
defaults will also be provided.  The Xen virt driver has limited its
configuration to be minimal, so the operators should only have a few required
options to set when Xen is selected as the virt driver.


Developer impact
----------------

The existing development team will be asked for reviews and approvals of the
change sets.  The Xen driver team will do the necessary implementation and
support of this function.


Dependencies
------------

 * `Multi-platform Host OS Enablement <https://blueprints.launchpad.net/openstack-ansible/+spec/multi-platform-host>`_
   - Needed to support Ubuntu 16.04 and CentOS 7


Implementation
==============

Assignee(s)
-----------

Primary assignee:
  Antony Messerli - antonym on IRC and Launchpad

Other contributors:

Work items
----------

Multiple changes would be needed:

* Add 'xen' to the 'nova_virt_types' structure alongside the necessary
  variable requirements for driver configuration, matching the other compute
  types.

* As required, add nova.conf templating for xen-specific configuration
  options that is conditionally included when nova_virt_type is 'xen'.

* Create a new nova_compute_xen.yml in the openstack-ansible-os_nova
  project.  This will contain the tasks needed to ensure the xen driver
  is installed and configured on the system.

* Update the existing nova_compute.yml to include the nova_compute_xen.yml
  and add the appropriate conditionals for that import.

* Create a new nova_compute_xen_install.yml, which will be included by
  nova_compute_xen.yml.  It will ensure that the necessary configuration
  and dependencies for running the Xen driver are in place.

* Update documentation and comments indicating the new Xen nova_virt_type
  and how to configure OpenStack-Ansible for the Xen driver.

* Automated unit test (see Testing)


Testing
=======

A new test-install-nova-xen.yml will be created for validating the new xen
playbooks within the openstack-ansible-os_nova project.


Documentation impact
====================

Documentation covering how to enable and configure Xen support will be
added to the user guide.


References
==========

Xen and OpenStack required versions: `<http://wiki.xenproject.org/wiki/OpenStack_CI_Loop_for_Xen-Libvirt>`_

Multi-platform Host OS Enablement: `<https://blueprints.launchpad.net/openstack-ansible/+spec/multi-platform-host>`_
