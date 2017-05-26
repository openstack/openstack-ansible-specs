Integration of OpenDaylight SDN controller with Neutron
#######################################################
:date: 2017-06-08 11:00
:tags: opendaylight, neutron, SDN, openvswitch

Blueprint on Launchpad:

  * https://blueprints.launchpad.net/openstack-ansible/+spec/opendaylight

This spec introduces the work required to have the OpenDaylight (ODL) SDN
controller deployed as a Neutron backend, using networking-odl ML2
mechanism driver to connect to Neutron.

This spec also covers the connection of OpenvSwitch (OvS) to OpenDaylight.

Problem description
===================

OpenStack networking (Neutron) uses a modular approach that allows different
backends to be used, by means of mechanism drivers. Although Neutron can handle
simple deployments, more advanced networking capabilities are better addressed
with an advanced SDN controller, such as OpenDaylight.

Proposed change
===============

The proposed change consists on using the existing OpenDaylight Ansible role
and optionally deploying it along with Neutron. The connection of OpenDaylight
and Neutron is done by using the networking-odl ML2 mechanism driver, and
configuring Neutron to use it.

After OpenDaylight and networking-odl are installed, some configuration is
required in ``ml2.ini`` file to instruct Neutron to use the mechanism driver,
as well as setup OpenDaylight's endpoint and credentials.

Final step consists on connecting the traffic forwarding elements with the
OpenDaylight controller. This spec will use OpenvSwitch for this task. Also,
neutron-openvswitch-agent needs to be stopped and disabled, as OpenDaylight
is the responsible for data plane management.

Alternatives
------------

There are other networking backend for Neutron already available with
Openstack-Ansible, namely Calico (https://www.projectcalico.org/tag/openstack/)
and DragonFlow (https://wiki.openstack.org/wiki/Dragonflow). These backends
are optionally deployed depending on the ML2 configuration that is passed to
``os_neutron`` Ansible role.

Playbook/Role impact
--------------------

The os_neutron role will be modified to optionally deploy OpenDaylight. The
proposal is to use the same approach as DragonFlow: when the
``neutron_plugin_type`` variable is set to ``ml2.opendaylight``, OpenDaylight
would be used as Neutron backend. There will be a new taskfile,
``neutron_opendaylight_setup.yml``, which would be included in os_neutron's
playbook when the above condition is fulfilled.

The OpenvSwitch scenario would be leveraged to work with OpenDaylight, being
OvS the only switch supported in the first release.

Upgrade impact
--------------

This is the first implementation of OpenDaylight with Openstack-Ansible, so no
upgrade concerns yet.

Security impact
---------------

Networking-odl configuration requires the setup of a username and password for
northbound auhentication towards OpenDaylight. The deployer should be able to
configure those credentials.

Communication between the controller and the switches will not be secured by
default. Using TLS to secure the communications is considered a stretch goal,
and deployers need to consider this security implication, specially in
production environments. For more information on secure communications between
OpenDaylight and OpenvSwitch, see the `References`_.

Performance impact
------------------

For those choosing to opt-in this deployment method, some extra packages need
to be installed on the system, which would make installation last a bit longer.

Extra resources are needed to run the OpenDaylight SDN controller on
the system as well. However, performance in Neutron API calls should be
minimum.

End user impact
---------------

End users would have a new networking API available through OpenDaylight. This
would enable them to create advanced networking scenarios (e.g. Service
Function Chaining). This will require some documentation with troubleshooting
steps to verify that OpenDaylight is working properly, as well as pointers
to OpenDaylight's official documentation.

No changes to Horizon or other OpenStack components are expected.

Deployer impact
---------------

New artifacts are being deployed, namely the Karaf runtime for OpenDaylight,
and the networking-odl pip package. OpenDaylight requires around 2.5G of RAM
to work properly, with OpenStack, that would need to be considered when
dimensioning the host where it will run.

Also deployers need to ensure that OpenvSwitch is deployed in all networking
nodes, namely compute hosts and hosts where neutron agents are running.

Developer impact
----------------

Developer impact is very low, all tasks for OpenDaylight deployment will be
optional and can be ignored when extending or modifying Neutron role.

Dependencies
------------

There are no dependencies

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  Juan Vidal (jvidal)

Other contributors:
  Fatih Degirmenci (fdegir)
  Daniel Farrell (dfarrell07)

Work items
----------

1. Install OpenDaylight SDN controller
2. Configure Neutron to use OpenDaylight
3. Deploy and configure OpenvSwitch to work with OpenDaylight
4. Set OpenDaylight as OpenvSwitch manager
5. Create a new test and verify that it passes
6. Document the new scenario

Testing
=======

As a replacement of Neutron backend, this new scenario should provide the same
capabilities of existing backends, so existing tests should be run.

A test specific for OpenDaylight can also be implemented, in the same way as
there are currently tests for Calico or DragonFlow.

Documentation impact
====================

The new scenario *OpenDaylight+OpenvSwitch* should be documented, explaining
the configuration parameters required to deploy it.

References
==========

Deploying OpenDaylight using Ansible:

* https://wiki.opendaylight.org/view/Deployment#Ansible_Role

Ansible role for OpenDaylight:

* https://git.opendaylight.org/gerrit/p/integration/packaging/ansible-opendaylight.git

Setting up OpenDaylight on OpenStack:

* https://wiki.opendaylight.org/view/OpenStack_and_OpenDaylight

Networking-odl mechanism driver:

* https://github.com/openstack/networking-odl

Networking-odl installation and configuration:

* https://docs.openstack.org/developer/networking-odl/installation.html

OpenvSwitch scenario with Openstack-Ansible:

* https://docs.openstack.org/developer/openstack-ansible-os_neutron/app-openvswitch.html

TLS Support on OpenDaylight OpenFlow plugin:

* https://wiki.opendaylight.org/view/OpenDaylight_OpenFlow_Plugin:_TLS_Support

Secure Communication Between OpenFlow Switches and Controllers

* https://www.thinkmind.org/download.php?articleid=afin_2015_2_30_40047
