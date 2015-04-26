Implement Ceilometer
####################
:date: 2015-03-31 10:30
:tags: kilo, ceilometer

This blueprint was created to add the Celiometer project to OSAD.
It will lay out a possible solution that will hopefully in turn
create discussion around how to properly implement ceilometer as an
OPTIONAL component of OSAD.

* https://blueprints.launchpad.net/openstack-ansible/+spec/implement-ceilometer

Problem description
===================

Currently, OSAD does not implement Ceilometer for various reasons. One being
the unstable nature of the project in production environments - causing
ever growing database tables resulting in extremely slow API quries.
However, some may still want to deploy Ceilometer and may prefer to
deal with the problem internally.

Furthermore, these issues should not discount the credibility of Ceilometer
as an OpenStack project, and as such, it should be implemented into OSAD.


Proposed change
===============
An additional role, os_ceilometer, would need to be added to handle the
installation/configuration of the ceilometer services. Also, additional
configuration directives would need to be added to other projects (such as
cinder, nova, glance, etc..) so that they will generate notifications.

For the database, the /etc/openstack_deploy/conf.d/ceilometer.yml should give
the user an option as to which database they want to connect to. This BP
does not implement the deployment of an additional database for Ceilometer,
but can be discussed as a potential addition.

Furthermore, new host groups will be added to the env.d/ceilometer.yml
file so that the user may specify which infra nodes to install the central
agents on , and which compute nodes to install the compute agent on. The
proposed changes to the this file will look something like::

    component_skel:
      ceilometer_agent_compute:
        belongs_to:
        - ceilometer_all
      ceilometer_agent_central:
        belongs_to:
        - ceilometer_all
      ceilometer_agent_notification:
        belongs_to:
        - ceilometer_all
      ceilometer_collector:
        belongs_to:
        - ceilometer_all
      ceilometer_alarm_evaluator:
        belongs_to:
        - ceilometer_all
      ceilometer_alarm_notifier:
        belongs_to:
        - ceilometer_all
      ceilometer_api:
        belongs_to:
        - ceilometer_all

    container_skel:
      ceilometer_api_container:
        belongs_to:
          - metering-infra_containers
        contains:
          - ceilometer_agent_central
          - ceilometer_agent_notification
          - ceilometer_collector
          - ceilometer_alarm_evaluator
          - ceilometer_alarm_notifier
          - ceilometer_api
        properties:
          service_name: ceilometer
          container_release: trusty
      metering-compute_container:
        belongs_to:
        - metering-compute_containers
        contains:
        - ceilometer_agent_compute
        properties:
          service_name: ceilometer
          container_release: trusty
          is_metal: true

    physical_skel:
      metering-compute_containers:
        belongs_to:
        - all_containers
      metering-compute_hosts:
        belongs_to:
        - hosts
      metering-infra_containers:
        belongs_to:
        - all_containers
      metering-infra_hosts:
        belongs_to:
        - hosts

Notable changes:
  * New ceilometer role
  * New ceilometer playbooks
  * A openstack_user_config.yml file in the openstack_deploy/conf.d/ directory
    specifically for ceilometer configurations.
  * Added vars in appropriate files
  * Haproxy config changes for ceilometer apis.

Alternatives
------------

Don't do it.


Playbook impact
---------------

The ceilometer component should be OPTIONAL. And thus have no effect on other
playbooks when chosen not to be run. However, when chosen to be run, other
configurations across different projects can be changed to allow
notifications.

Upgrade impact
--------------

None known

Security impact
---------------

Credentials for the Ceilometer database may need to be specified by the user,
as this BP does not implement the deployment of the Ceilometer database at
this moment.

Performance impact
------------------

When ceilometer is enabled, the horizon dashboard can potentially slow down
due to large API responses from ceilometer. This is related to the problem
stated in the Problem Description. We must let the user know about this known
problem, and advise the ceilometer database be properly supervised.
When ceilometer is disabled, it should have no performance impact.


End user impact
---------------

The user will now have the option to enable ceilometer on their private cloud.

Deployer impact
---------------

Additional configs must be specified in
/etc/openstack_deploy/conf.d/ceilometer.yml only if the deployer wants
ceilometer to be enabled.

The deployer needs to be aware of the database that is going to be used
for ceilometer.

Developer impact
----------------

A new role will be created to handle the installation/configuration of
ceilometer. This role will be run explicitly by the deployer.

Dependencies
------------

N/A

Implementation
==============

Assignee(s)
-----------

Primary Assignee(s)
-------------------

Miguel Alejandro Cantu
Sudarshan Acharya

Other contributors:
-------------------


Work items
----------

At the very least, the following need to get done:
  * New ceilometer role
  * New ceilometer playbooks
  * A openstack_user_config.yml file in the openstack_deploy/conf.d/
    directory specifically for ceilometer configurations.
  * Added vars in the appropriate sections.
  * Haproxy config changes for ceilometer apis.

The following files need to be modified:

  * etc/openstack_deploy/env.d/openstack_environment.yml
  * etc/openstack_deploy/user_secrets.yml
  * etc/openstack_deploy/user_variables.yml
  * playbooks/inventory/group_vars/all.yml
  * playbooks/vars/configs/haproxy_config.yml
  * playbooks/vars/repo_packages/openstack_services.yml
  * playbooks/roles/os_cinder/defaults/main.yml
  * playbooks/roles/os_cinder/templates/cinder.conf.j2
  * playbooks/roles/os_glance/defaults/main.yml
  * playbooks/roles/os_glance/templates/glance-api.conf.j2
  * playbooks/roles/os_glance/tempaltes/glance-registry.conf.j2
  * playbooks/roles/os_heat/defaults/main.yml
  * playbooks/roles/os_heat/templates/heat.conf.j2
  * playbooks/roles/os_nova/defaults/main.yml
  * playbooks/roles/os_nova/templates/nova.conf.j2
  * playbooks/roles/os_swift/defaults/main.yml
  * playbooks/roles/os_swift/tasks/swift_service_setup.yml
  * playbooks/roles/os_swift/templates/proxy-server.conf.j2

The following files need to added:

  * etc/openstack/deploy/conf.d/ceilometer.yml.example
  * playbooks/os-ceilometer-install.yml
  * playbooks/roles/os_ceilometer/

    * CONTRIBUTING.rst
    * LICENSE
    * README.rst
    * defaults/

      * main.yml
    * handlers/

      * main.yml

    * meta/

      * main.yml

    * templates/

      * ceilometer.conf.j2
      * api_paste.ini.j2
      * ceilometer-upstart-init.j2
      * event_definitions.yaml.j2
      * event_pipeline.yaml.j2
      * pipeline.yaml.j2
      * sudoers.j2

    * tasks/

      * main.yml
      * ceilometer_install.yml
      * ceilometer_pre_install.yml
      * ceilometer_post_install.yml
      * ceilometer_service_setup.yml
      * ceilometer_service_add.yml
      * ceilometer_upstart_common_init.yml
      * ceilometer_upstart_init.yml


Testing
=======
The gate scripts will need to be modified with the ceilometer
configurations turned on. A variable of the like of
"DEPLOY_CEILOMETER" will be added with the appropriate
conditionals in place to deploy ceilometer with a mongodb
server for testing.

The playbooks will point to a mongodb server deployed on the AIO.
A gate script will be created to deploy a simple mongodb server on the AIO
using an ansible galaxy role.


Documentation impact
====================

Thorough documentation will need to be created explaining how to enable
ceilometer.


References
==========

* http://goo.gl/LA7o6N
* http://goo.gl/Xj7cqT
* https://www.rdoproject.org/CeilometerQuickStart
* http://docs.openstack.org/developer/ceilometer/install/manual.html
