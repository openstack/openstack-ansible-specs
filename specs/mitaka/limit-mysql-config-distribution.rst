Limit Mysql Config Distribution
###############################
:date: 2015-07-20
:tags: mysql, galera


* https://blueprints.launchpad.net/openstack-ansible/+spec/limit-mysql-config-distribution


Problem description
===================

The distribution of the ``.my.cnf`` file should be limited to API nodes and the
utility container.


Proposed change
===============

* Add a variable to the the galera_client role to limit the distribution of the ``.my.cnf``
  file.


Alternatives
------------

Leave everything the way it is.


Playbook/Role impact
--------------------

This will change the galera_client and "os_*" roles to ensure that the ``.my.cnf``
files are only distributed to a limited set of hosts.


Upgrade impact
--------------

n/a


Security impact
---------------

By limiting the distribution of the ``.my.cnf`` file we should be able to improve general
system security.


Performance impact
------------------

n/a


End user impact
---------------

n/a


Deployer impact
---------------

n/a


Developer impact
----------------

n/a


Dependencies
------------

n/a

Implementation
==============

Assignee(s)
-----------

Primary assignee: (unassigned)


Work items
----------

* Add a variable to the galera_client role to disable the task "Drop local .my.cnf file"
* Change the meta entries where the **galera_client** roles is used use the new variable
  where appropriate.


Testing
=======

This will be tested within every gate check for functionality.


Documentation impact
====================

n/a


References
==========

Bug reference for the change:
  * https://bugs.launchpad.net/openstack-ansible/trunk/+bug/1412393
