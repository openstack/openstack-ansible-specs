Use nginx as centralized reverse proxy for API services
#######################################################
:date: 2017-12-02 00:00
:tags: nginx, load balancing, wsgi, API

In the previous cycle, most OpenStack API services provided a WSGI application
which is being serviced through uWSGI. The aim of this spec is to outline a
plan for taking advantage of the greater flexibility uWSGI provides and making
use of nginx as a centralized reverse proxy.

https://blueprints.launchpad.net/openstack-ansible/+spec/centralized-nginx

Problem description
===================

Within most roles, OpenStack API services are now deployed and served through
uWSGI. Some however, also include the installation of a web server proxy in
front of that. The web server is currently assumed to be installed on the
host in isolation and managed through the installing role. This causes issues
with converged installations, such as an all bare metal scenario. SSL
encryption of service requests is also currently difficult because of this
inconsistency between individual service deployments. For the most part, SSL
termination in OpenStack-Ansible deployments is expected to be handled at the
load balancer but deployers may require additional controls over the handling
of encrypted traffic.

Proposed change
===============

Remove the installation of nginx from any OpenStack role that includes it
today and instead deploy it separately as a shared reverse proxy with
individual sites configured for each OpenStack API service. The OpenStack roles
will only need to provide site configurations for an existing nginx installation.

For management of the uWSGI backends that nginx will be proxying, install a uWSGI
FastRouter alongside nginx. The FastRouter will create a shared socket that
nginx can connect to, and a subscription server for the applications to subscribe
to and automatically provide load balancing for.

Alternatives
------------

nginx could instead be deployed alongside each OpenStack service, but that
could have an effect on performance due to the additional processes running on
the host and wouldn't address the duplication of tasks and conflicts caused
when multiple roles are attempting to manage the same web server on the same
physical host or container.

The load balancer backends could use uWSGI http/https directly, most projects
recommended a dedicated web server however. With the proposed change, there is
a clear separation between the handling of http requests and running of Python
code. Load balancer configurations can also be simplified since all OpenStack
API backends would use the same set of nginx hosts, instead of requiring knowledge
of individual containers. And for scaling purposes, a new or replaced API service
installation will only need to subscribe to the FastRouter to be included in the
pool.

Playbook/Role impact
--------------------

A new playbook will need to be created to install nginx and a uWSGI FastRouter.
An existing nginx role from galaxy would be preferred, but one may need to be
created or contributed to if there is not one that supports all of the same
operating systems that OpenStack-Ansible does or that does not provide
sufficient configuration options for our needs.

Each existing OpenStack role that deploys a web server will need to have those
tasks removed and replaced with a task to configure and enable an nginx site,
delegated to any hosts running nginx. The uWSGI configuration within those
roles will also need to be updated with options to subscribe to an existing
FastRouter.

The HAProxy server role may require more flexibility, such as allowing SSL
passthrough and just-in-time configuration of services and backends.

Upgrade impact
--------------

Since load balancer backends will likely be changing, the load balancer will
need to initially contain both the existing backends and the centralized nginx
backends to limit downtime during upgrades.

Security impact
---------------

The proposed change should increase the security posture of OpenStack-Ansible
since it will allow deployers to have more flexibility over where SSL is
terminated, including preventing traffic from leaving a host once it is
unencrypted.

Performance impact
------------------

nginx is a high performance web server and reverse proxy. There may be some
host performance increase with less web servers running on a single controller
host. Deployment times may also be improved by removing the duplicated tasks of
installing and configuring a web server within multiple roles.

End user impact
---------------

N/A

Deployer impact
---------------

Deployers will need to be aware of the deployment architecture changes this
change includes. Additional options will be available to deployers to configure
a shared nginx instance, uWSGI FastRouter, and the distribution of SSL
certificates. Deployers must also be made aware of any upgrade impact this
change may have.

Developer impact
----------------

Future OpenStack roles will need to include the patterns established by this
change. Tests will require additional Ansible roles, playbooks, and variables.

Dependencies
------------

The HAProxy server role may require some changes and increased configuration
flexibility.

The keystone role currently conditionally installs Apache when used as an
identity provider or service provider for an external identity provider. This
will need to be investigated further to ensure that nginx can provide the same
level of support for those roles, including gated test scenarios.

The horizon role also currently installs Apache and uses it to serve static
content. When running on a container, that static content may need to move to a
mount to remain accessible to an external web server.

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  jimmy-mccrory (jmccrory)

Other contributors:
  <launchpad-id or None>

Work items
----------

* Evaluate existing nginx roles in galaxy
* Develop new nginx role if necessary
* Develop playbook for deploying nginx and uWSGI FastRouter
* Adapt HAProxy role
* Evaluate bind mounting of files statically served by web server
* Update OpenStack roles to create nginx site configuration and subscribe to
  FastRouter for API services

Testing
=======

Individual roles and the integrated repo will test the changes involved as they
are implemented.

Documentation impact
====================

The changes to the deployment architecture and any additional options for
configuring nginx, uWSGI FastRouter, HAProxy, and SSL will need to be
documented.

The impacts to upgrades and steps to minimize, and hopefully avoid, API
downtime will also need to be documented.

References
==========

http://uwsgi-docs.readthedocs.io/en/latest/Fastrouter.html
