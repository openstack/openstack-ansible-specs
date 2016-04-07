Support PowerVM Virt Driver
###########################
:date: 2016-03-18 14:45
:tags: ansible, powervm

The purpose of this spec is to add support for the PowerVM compute platform to
OpenStack-Ansible. This will enable deployment of PowerVM systems as OpenStack
compute nodes alongside the core OpenStack components.


https://blueprints.launchpad.net/openstack-ansible/+spec/powervm-virt-driver


Problem description
===================

The PowerVM Compute Driver[1] is currently an out of tree, OpenStack compliant
Nova Driver.  The Nova compute team has asked for us to grow our usage before
inclusion into the main Nova project.  Our potential users have cited to us
that they need OpenStack-Ansible support to bring PowerVM into their
environments.

Bringing this support for the PowerVM driver expands the number of operators
that will make use of the compute driver.  It also helps grow the usage so that
it meets the requirements from the Nova team for inclusion into the main source
tree.

Openstack-Ansible supports provisioning kvm/qemu compute nodes, and is
introducing support for other virt driver types such as Ironic.  This
blueprint would add support for the PowerVM Nova Virt Driver.


Proposed change
===============

The PowerVM platform runs virtualization and management resources in a VM.
This privileged VM has authority to manage the system and is where the
nova-compute driver runs.  It currently supports running on Ubuntu 15.10[2],
and will run on Ubuntu 16.04 in the Newton timeframe.

The PowerVM compute driver can be paired with the standard OVS Agent today,
with support for the Linux Bridge network agent planned for the Newton
release. This blueprint covers deploying the nova-powervm compute driver with
the standard networking agents that OpenStack-Ansible supports.

The proposed changes include:
* Add support for installing/configuring the PowerVM virt driver and dependencies
* Tests to verify changes to the os_nova role required for PowerVM support

Note: PowerVM also supports a platform-specific 'Shared Ethernet' ML2 Agent,
which is not covered in this blueprint.


Alternatives
------------

* Maintain independent PowerVM Ansible playbooks - This requires reinvention
  of base function and does not meet operator requirements.


Playbook/Role impact
--------------------

See the Work Items for the playback/role impact.  There will be a new
nova-compute tag of 'nova-powervm' that operators would use to support the
PowerVM compute driver, and references to nova_virt_type will be updated to
reflect a 'powervm' option.


Upgrade impact
--------------

None. The nova-powervm driver is new for OpenStack-Ansible, and as such has
no upgrade impact.


Security impact
---------------

None.


Performance impact
------------------

None.


End user impact
---------------

Deployers will be able to deploy compute nodes with the PowerVM virt driver.


Deployer impact
---------------

PowerVM specific configuration options will be added to the OpenStack-Ansible
os_nova role. When support for PowerVM as a virt driver is enabled these
config options will be used during deploy; however it is expected PowerVM
support will be disabled by default, requiring that deployers explicitly
enable PowerVM support and configure hosts for openstack-ansible to use.

Documentation of these new configuration items will be provided and a set of
defaults will also be provided.  The PowerVM driver has limited its
configuration to be minimal, so the operators should only have a few required
options to set when PowerVM is selected as the virt driver.


Developer impact
----------------

The existing development team will be asked for reviews and approvals of the
change sets.  The PowerVM driver team will do the necessary implementation and
support of this function.


Dependencies
------------

 * Ironic nova_virt_type enhancements [4] - (Merged) This introduces support
   for additional nova_virt_types, which this work will expand on.
 * Open vSwitch agent support [5] - Soft dependency on this to introduce OVS
   support, which the PowerVM computer driver supports today.
 * Ubuntu 16.04/Multi-Host Support [6] - Needed to support Ubuntu 16.04, which
   will be the target OS for the PowerVM driver in the Newton timeframe.


Implementation
==============

Assignee(s)
-----------

Primary assignee:
  Wang Qing wu - wangqwsh on IRC and Launchpad

Other contributors:
  Drew Thorstensen - thorst on IRC and Launchpad
  Adam Reznechek - adreznec on IRC and Launchpad


Work items
----------

Multiple changes would be needed:

* Update the openstack_other.yml in the main openstack-ansible project to
  include the nova-powervm project.

* Define a new tag called nova-powervm.  This will be used across the
  openstack-ansible projects.

* Add 'powervm' to the 'nova_virt_types' structure alongside the necessary
  variable requirements for driver configuration, matching the other compute
  types.

* As required, add nova.conf templating for powervm-specific configuration
  options that is conditionally included when nova_virt_type is 'powervm'.

* Create a new nova_compute_powervm.yml in the openstack-ansible-os_nova
  project.  This will contain the tasks needed to ensure the powervm driver
  is installed and configured on the system.

* Update the existing nova_compute.yml to include the nova_compute_powervm.yml
  and add the appropriate conditionals for that import.

* Create a new nova_compute_powervm_install.yml, which will be included by
  nova_compute_powervm.yml.  It will ensure that the necessary configuration
  and dependencies for running the PowerVM driver are in place.

* Update documentation and comments indicating the new PowerVM nova_virt_type
  and how to configure OpenStack-Ansible for the PowerVM driver.

* Automated unit test (see Testing)


Testing
=======

The PowerVM Driver CI System is currently using devstack for its set up.  This
cloud will be updated to make use of OpenStack-Ansible to deploy the
operator cloud that runs the CI infrastructure.

A new test-install-nova-powervm.yml will be created for validating the new
powervm playbooks within the openstack-ansible-os_nova project.


Documentation impact
====================

Documentation covering how to enable and configure PowerVM support will be
added to the user guide.


References
==========

1. nova-powervm driver: https://github.com/openstack/nova-powervm
2. PowerVM NovaLink: https://www.ibm.com/developerworks/community/wikis/home?lang=en#!/wiki/Power%20Systems/page/Introducing%20PowerVM%20NovaLink
3. PowerVM Mitaka Update: https://www.ibm.com/developerworks/community/wikis/home?lang=en#!/wiki/Power%20Systems/page/OpenStack%20and%20PowerVM%20-%20Mitaka%20Update
4. Nova config for os_ironic: https://review.openstack.org/#/c/293315
5. Neutron Open vSwitch Agent support: https://review.openstack.org/#/c/298765/
6. Support Ubuntu 16.04: https://blueprints.launchpad.net/openstack-ansible/+spec/multi-platform-host
