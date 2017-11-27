OpenDaylight with BGPVPN support in Neutron
###########################################
:date: 2017-11-17 16:30
:tags: OpenDaylight,Open vSwitch,neutron,BGPVPN,L3,DC-GW

Blueprint on Launchpad

  * https://blueprints.launchpad.net/openstack-ansible/+spec/opendaylight-with-bgpvpn-support


This spec introduces the work required for OpenDaylight configured with BGPVPN
through Openstack-Ansible to enable Openstack deployments with extended L3 support.

Problem description
===================

The support for BGPVPN is available from OpenDaylight since its Beryllium
release. Openstack can make use of this feature by configuring neutron to use
BGPVPN service plugin.

 `` https://docs.openstack.org/networking-bgpvpn/latest/user/drivers/opendaylight/index.html ``
 `` https://docs.openstack.org/networking-bgpvpn/latest/user/usage.html ``

In addition to it, quagga/zrpcd and its dependent packages have to be installed
along with OpenDaylight for configuring OpenDaylight as a BGP speaker.

Proposed change
===============

For the configuration of OpenDaylight as a BGP speaker that integrate into
deployer's infrastructure, a new OpenStack-Ansible playbook with required
ansible tasks for installing quagga and its required packages will be written.
The wiring of the OpenDaylight configuration as a BGP speaker will be done
inside the neutron role, which configures OpenDaylight (see playbook/role
impact for details).

The initial supported distros would be CentOS and Ubuntu.

Alternatives
============

There are other bgpvpn backend drivers available with neutron like BaGPipe,
OpenContrail driver and Nuage Network driver to configure the BGPVPN.

Playbook/Role impact
--------------------

The new playbook will be added in OpenStack-Ansible which installs quagga and
configure OpenDaylight for BGP speaker. This playbook would get executed after
neutron playbook in neutron server node (in case of ha deployment, among three
neutron server containers, one is chosen), because quagga just needs to get
installed in one of the OpenDaylight node and run additional karaf CLI
commands to make it as BGP speaker.

The proposal is to add a extra variable in neutron_plugin_base, overriding the
default ODL behavior, and trigger the usage of BGPVP.
When ``neutron_plugin_type`` variable set to ``ml2.opendaylight``,
``neutron_plugin_base`` list variable having
`networking_bgpvpn.neutron.services.plugin.BGPVPNPlugin`` item, then neutron
server node will be installed/configured with OpenDaylight and Quagga.

Upgrade impact
--------------

This is the first implementation of OpenDaylight with Quagga, so no
upgrade concerns yet.

Security impact
---------------

Networking-bgpvpn configuration requires the setup of a username and password for
northbound authentication towards OpenDaylight. The deployer should be able to
configure those credentials.

Communication between the controller and the switches will not be secured by
default. Using TLS to secure the communications is considered a stretch goal,
and deployers need to consider this security implication, specially in
production environments. For more information on secure communications between
OpenDaylight and OpenvSwitch, see the `References`_.

Performance impact
------------------

For those choosing to opt-in this deployment method, some extra packages need
to be installed on the neutron server, which would make installation last a
bit longer.

Extra resources are needed to run the OpenDaylight SDN controller on
the system as well. However, performance in Neutron API calls should be
minimum.

End user impact
---------------

End users would have a new networking and BGPVPN API available through Neutron.
This would enable them to create bgpvpn scenarios (e.g. Router and Network
association with BGPVPN). This will require some documentation with troubleshooting
steps to verify that OpenDaylight is working properly, as well as pointers
to OpenDaylight's official documentation.

No changes to Horizon or other OpenStack components are expected.

Deployer impact
---------------

New artifacts are being deployed, namely the Karaf runtime for OpenDaylight,
quagga/zrpcd, thrift and the networking-odl pip package. OpenDaylight requires
around 2.5G of RAM to work properly, with OpenStack, that would need to be
considered when dimensioning the host where it will run.

Also deployers need to ensure that OpenvSwitch with version >= 2.8 is deployed
in all networking nodes, namely compute hosts and hosts where neutron agents are
running.

Developer impact
----------------

Developers have a new playbook to maintain, whose scope is very reduced and not
in the path of all deployments.

Developer impact is very low, all tasks for BGPVPN deployment will be optional
and can be ignored.
The tasks won't be skipped, but instead no host will be matched for the new
playbooks. This way, if we put the playbook on the path for every
developer/deployer, the impact will be minimum.


Dependencies
------------

There are no dependencies

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  Periyasamy Palanisamy (epalper)
  Dimitrios Markou (mardim)

Work items
----------

1. Add new playbook for installing/configuring quagga/zrpcd
2. Task to configure ODL as a BGP speaker
3. Make neutron role to get configured with OpenDaylight BGPVPN driver
4. Create a new test and verify that it passes
5. Document the new functionality

Testing
=======

As a replacement of Neutron backend, this new scenario should provide the same
capabilities of existing backends, so existing tests should be run.

A test specific for OpenDaylight can also be implemented, in the same way as
there are currently tests for Calico or DragonFlow.

Documentation impact
====================

The new scenario *OpenDaylight+BGPVPN* will be documented, explaining
the configuration parameters required to deploy it.

References
==========

OpenDaylight scenario with OpenStack-Ansible

* https://docs.openstack.org/openstack-ansible-os_neutron/latest/app-opendaylight.html
* https://git.openstack.org/cgit/openstack/openstack-ansible-specs/tree/specs/pike/opendaylight.rst

packaging and installing quagga/zrpcd packages

* https://github.com/opnfv/apex/blob/master/build/build_quagga.sh

BGP peering with OpenDaylight

* https://github.com/opnfv/sdnvpn/blob/master/sdnvpn/test/functest/testcase_3.py

Enabling BGPVPN mechanism driver at neutron

* https://docs.openstack.org/networking-bgpvpn/latest/user/drivers/opendaylight/index.html
