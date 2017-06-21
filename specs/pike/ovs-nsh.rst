Openvswitch with NSH support in Neutron
#######################################
:date: 2017-06-21 15:00
:tags: Openvswitch,neutron,SFC,NSH

Blueprint on Launchpad

  * https://blueprints.launchpad.net/openstack-ansible/+spec/openvswitch-with-nsh-support


This spec introduces the work required to have Open vSwitch with NSH protocol
support which is used in Service Function Chaining.

Problem description
===================

According to * https://datatracker.ietf.org/doc/draft-ietf-sfc-nsh/
Network Service Header (NSH) is inserted to a packet or a frame to realize
service functions paths. Also provides a mechanism for metadata exchange
along the instantiated service path. The NSH protocol is used as an SFC
encapsulation which is required for the support of the Service Function
Chaining (SFC) Architecture as it defined in RFC7665.

The Openvswitch currently doesn't support the NSH protocol. So the only way
to add NSH support to Open vSwitch is through Yi Yang's patches
(https://github.com/yyang13/ovs_nsh_patches).

Proposed change
===============

The proposed change is the use of the existing Neutron Ansible Role
for the installation of Open vSwitch with NSH support when the user selects
that functionality through specific configuration in Openstack-Ansible
project. We intent to configure only Neutron component and not use the
aforementioned functionality for end to end testing.

The installation of Open vSwitch with NSH support will be addressed
by the use of specific packages which are going to be maintained in
private repositories unti the NSH functionality will be included in a
subsequent release of Open vSwitch project.


Alternatives
============

An alternative to create a SFC without NSH is the port chaining technique.
The aforementioned technique uses Neutron ports to steer the traffic to a
service chain and has no notion of the actual services which are attached
to those Neutron ports.

Playbook/Role impact
--------------------

The os_neutron role will be modified to optionally install Open vSwitch with
NSH support. The proposal is to add an extra variable so the user can decide
whether or not he needs to add NSH support with the Open vSwitch installation. When
the ``neutron_plugin_type`` variable is set to ``ml2.ovs`` or ``ml2.dragonflow``
and the ``ovs_nsh_support`` variable is set to ``true`` then the Open vSwitch will
be installed with NSH support. So there will be an extra task in the ``neutron_pre_install.yml``
which will add the distribution specific repositories with the ovs_nsh packages.

Upgrade impact
--------------

This is the first implementation of Open vSwitch with NSH support in OpenStack-Ansible,so
no upgrade concerns yet.

Security impact
---------------

No security impact

Performance impact
------------------

The added NSH support to Open vSwitch will not have any performance impact to the current
OpenStack-Ansible installation because the system will need to install only some extra packages.

End user impact
---------------

The end users will have the capability to create service function chains with the use
of the NSH protocol.  Also they can use OpenDaylight as networking backend which
via the ``sfc`` component supports the creation of SFCs through the NSH protocol.

Deployer impact
---------------

The deployer needs to ensure that the specific repositories which hold the ovs_nsh packages
are added to the system and the proper Open vSwitch packages are installed.

Developer impact
----------------

The developer impact is really low because the NSH support for Open vSwitch is optional
and can be ignored when extending or modifying Neutron role.

Dependencies
------------

There are no dependencies

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  Dimitrios Markou (mardim)

Work items
----------

1. Add specific PPA for ovs_nsh packages
2. Install Open vSwitch with NSH protocol suppport
3. Document the new functionality

Testing
=======

Existing tests should be run because the only thing that change is that the
installation of Open vSwitch is managed by specific repositories when NSH support is selected.

Documentation impact
====================

The new functionality *Open vSwitch with NSH support* should be documented, explaining
the required configuration parameters which are necessary for this deployment.

References
==========

Open vSwitch scenario with OpenStack-Ansible

* https://docs.openstack.org/openstack-ansible-os_neutron/latest/app-openvswitch.html

NSH ietf draft

* https://datatracker.ietf.org/doc/draft-ietf-sfc-nsh/

SFC RFC 7665

* https://tools.ietf.org/html/rfc7665

PPA for Openvswitch-NSH packages

* https://launchpad.net/~mardim/+archive/ubuntu/mardim-ppa

Openvswitch-NSH packages for Centos

* https://copr.fedorainfracloud.org/coprs/mardim/openvswitch-nsh/
