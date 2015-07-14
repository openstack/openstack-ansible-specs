Multi-Region Swift
####################
:date: 2015-07-03 13:00
:tags: kilo, swift

This blueprint was created to add Multi-Region Swift support to OSAD. It will
lay out a series of use cases to define the requirements of Multi-Region Swift
within OSAD.

* https://blueprints.launchpad.net/openstack-ansible/+spec/multi-region-swift

A Swift cluster can be deployed in such a way that the cluster spans multiple
geographically dispersed data centers. This allows an end-user to ensure
resiliency in the event of a data center failure, with one or more copies of
an object stored in each data center location. This facilitates end-users
being able to build out geographically dispersed infrastructure, enabling high
availability.

Problem description
===================

* As a User I want to be able to store or retrieve objects in any location of a
  multi-location object storage solution, using the same credentials.

* As a User, I want my default object storage/retrieval location to be as close
  to my location as possible - by specifying the closest endpoint and having
  that endpoint access the closest storage locations.

* As an Administrator, I want to be able to set a global storage policy that
  enables me to specify the number of copies of every object to be stored in
  each location of the multi-location object storage based on one of the
  following predefined scenarios:

        - With 3 or more storage deployments, each location is considered to be
          as important as any other location. The same number of copies should
          be kept in each location.

        - With 3 storage deployments we consider 1 to be the primary and the
          other 2 as geographically convenient locations for read purposes.
          There should be 2 copies in the primary location and 1 in each of
          the other locations.

        - With 2 storage deployments, we consider 1 to be the primary location
          and the other a "backup" location. There should be 2 copies in the
          primary location and 1 copy in the "backup" location.

* As a User, I want to be assured durability of my content under various
  circumstances, such as:

        - Initial upload to single location.  i.e. in the case of uploading
          an object to location A in a 3 location solution where the global
          policy is 1 locally, 1 in each remote - until the 2 remote objects
          are confirmed, the local object storage cluster will have 3 copies.

        - Failure of 1 or more locations.  i.e. In a 3 location solution
          where the global policy is 1 locally, 1 in each remote - if
          location A fails, either location B or C will generate a second copy
          of the 'missing' objects to ensure that there are always 3 copies.

* As an Administrator, I want to be able to override the global storage policy
  at a container level in order to increase or reduce replication. For example:

        - One or more containers can have an alternate policy specified at
          the container level that overrides the global policy. If the global
          policy in a 3 location solution is 1 locally and 1 in each remote,
          one or more container can be configured to a 3 locally policy.

* As a User, where the global storage policy has been configured to replicate
  objects across locations, I want to be able to retrieve my objects from an
  alternate location in the event of a failure of my default location.

* As an Administrator, I want to be assured that when replicating data from
  the primary location to the remote locations the data is secured and not
  transmitted in the clear.


Proposed change
===============

1. Enable the use of the read_affinity, write_affinity and
   write_affinity_node_count settings within Swift on a swift-proxy host basis.
   This will allow the prioritization of reads/writes based on region.

2. Enable some form of encryption to ensure the replication of objects across
   locations is secure.

3. Configure the management of the ring and keys required for communication
   between swift storage hosts across multiple locations and "deployments".

4. Document the use of the read_affinity, write_affinity and
   write_affinity_node_count settings within os-ansible-deployment and provide
   guidance around how to implement the specified Use Cases.

5. Document the process of setting up a global swift cluster within
   os-ansible-deployment.

6. Adjust settings to allow the use of keystone v3 API for swift, in the swift
   configuration files.

Alternatives
------------

We could not enable swift multi-region support.


Playbook impact
---------------

Whilst the Multi-Region component will be optional, we will need to implement the
following changes without adjusting how the current default operates:

1. The Multi-Region component will require adjustments to ring/key management
   for swift hosts, as well as some changes to how the Swift inventory is
   managed within the user_config.yml (or conf.d/swift.yml) files.

2. Read_affinity, write_affinity and write_affinity_node_count will need to be
   added configuration for proxy-servers.

3. The synchronization of the swift-ring will needed to be handled across nodes
   in all locations/regions.

Upgrade impact
--------------

This should have no upgrade impact.

Security impact
---------------

Since swift does not handle encryption of objects this will need to be handled
externally to swift.

Performance impact
------------------

N/A

End user impact
---------------

The user will now have the option to configure a Multi-Region Swift cluster.
The default will remain the same, so it should not impact any users who do not
wish to utilise a Multi-Region Swift cluster.

Deployer impact
---------------

A deployer will be able to adjust the inventory across multiple deploys to
ensure a global swift cluster operating unformly for all deploys.

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
  https://launchpad.net/~andrew-mccrae (andymccr)

Other contributors:
  https://launchpad.net/~steve-lewis (stevelle)
  https://launchpad.net/~tom-cameron (rackertom)
  https://launchpad.net/~apsu-2 (Apsu)
  https://launchpad.net/~prometheanfire (prometheanfire)


Work items
----------

1. Enable the use of the read_affinity, write_affinity and
   write_affinity_node_count settings within Swift on a swift-proxy host basis.
   This will allow the prioritization of reads/writes based on region.

2. Enable some form of encryption to ensure the replication of objects across
   locations is secure.

3. Configure the management of the ring and keys required for communication
   between swift storage hosts across multiple locations and "deployments".

4. Document the use of the read_affinity, write_affinity and
   write_affinity_node_count settings within os-ansible-deployment and provide
   guidance around how to implement the specified Use Cases.

5. Document the process of setting up a global swift cluster within
   os-ansible-deployment.

6. Adjust settings to allow the use of keystone v3 API.

Testing
=======

As this will require two independent installations of swift we won't add anything
specific to the gate to automatically test this. However the changes should not
adjust how current tests work and all changes will need to ensure that existing
tests continue to pass.

Documentation impact
====================

1. Use case implementation will need to be documented

2. Implementation of a global cluster and the settings required.

3. New network requirements will need to be documented.

4. Inventory management, and configuration options that are added as a result
   will need to be documented.

References
==========

* http://docs.openstack.org/developer/swift/admin_guide.html#geographically-distributed-clusters

* https://swiftstack.com/blog/2012/09/16/globally-distributed-openstack-swift-cluster/
