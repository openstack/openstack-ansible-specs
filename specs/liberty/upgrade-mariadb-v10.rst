MariaDB upgrade to v10
######################
:date: 2015-07-19
:tags: mysql, galera

The purpose of this spec is to upgrade MariaDB from v5.5 to v10.0

https://blueprints.launchpad.net/openstack-ansible/+spec/MariaDB-upgrade-to-v10


Problem description
===================

MariaDB + Galera is presently using v5.5 which is old and should be upgraded.
Additionally, we are using xtrabackup v1 which was deprecated in favor of xtrabackup
v2 as such that should be changed as we upgrade to v10 so that we can take advantage
of the performance and security enhancement available in the new releases.


Proposed change
===============

* Upgrade MariaDB - this is a package change as well as upstream mariadb repo
  change
* Change xtrabackup to xtrabackup-v2 - This will add a configuration section in
  the default ``my.cnf`` for the xtrabackup client(s).


Alternatives
------------

Leave everything the way it is.


Playbook/Role impact
--------------------

There will be no playbook impact however the The galera_server and galera_client
roles will change to support the new packages for xtrabackup-v2 and mariadb+galera
v10.


Upgrade impact
--------------

n/a


Security impact
---------------

Upgrading to MariaDB v10 w/ xtrabackup v2 will result OSAD being able to take
advantage of better security options in the future if we so choose.


Performance impact
------------------

Upgrading to MariaDB v10 w/ xtrabackup v2 will result in greater performance.


End user impact
---------------

n/a


Deployer impact
---------------

The deployer will need to be aware that mariadb v5.5 is being upgraded however
all of the post upgrade processes should be handled automatically.


Developer impact
----------------

n/a


Dependencies
------------

* SPEC/Limit the distribution of .my.cnf - https://review.openstack.org/#/c/203754/


Implementation
==============

Assignee(s)
-----------

Primary assignee: (unassigned)


Work items
----------

* Change the package for MariaDB10 w/ Galera
* Add repo for new versions of XtraBackup
* Update the my.cnf for use with MariaDB10 (revise it for anything that may
  need to be removed)
* Update the cluster.cnf for use with MariaDB10 (revise it for anything that
  may need to be removed)


Testing
=======

The testing for this change will be automatic in upstream as everybuild will
change to using this by default.


Documentation impact
====================

n/a


References
==========

n/a
