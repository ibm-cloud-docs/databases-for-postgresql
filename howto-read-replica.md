---

Copyright:
  years: 2019
lastupdated: "2019-11-01"

keywords: postgresql, databases, replica

subcollection: databases-for-postgresql

---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}
{:tip: .tip}

# Configuring Read-only Replicas
{: #read-only-replicas}

You can set up your {{site.data.keyword.databases-for-postgresql_full}} deployment to be a read-only replica of another {{site.data.keyword.databases-for-postgresql}} deployment. 

A read-only replica is set up to replicate all of your data from the leader deployment to the replica deployment using asynchronous replication. As the name implies, read-only replicas support read transactions and can be used to balance databases that have both write-heavy and read-heavy operations. The read-only replica has a single PostgreSQL data member, and it is billed at the [same per member consumption rates as the leader](https://{DomainName}/catalog/services/databases-for-postgresql/).

## Read-only Replica Considerations

- The read-only replica can exist in the same region as the source formation or in different one, enabling your data to be replicated across regions.

- A read-only replica must be the same major version as its leader. 

- Backups are disabled on read-only replicas. Backups are taken only on leader deployments.

- Read replication is not supported into or out of EU Cloud-enabled regions (currently `eu-de`). It is supported within those regions.

- There is a limit of 5 read-only replicas per leader.

- The read-only replica does not participate in master/follower elections for the leader cluster and failover to the read-only replica is not automated. Promotion of the read-only replica to a full deployment is a manual, user-initiated task.

- The minimum size of a read-only replica is 2 GB RAM and 10 GB of disk. This is true even if your leader deployment is smaller.

- Read-only replicas do not auto-scale to match the leader. If the amount of data you store outgrows the disk that is allocated to your deployments, scale the disk on the read-only replicas and then the leader. Scaling the read-only replica first ensures that you do not run out of space on the read-only replicas. If you have scaled the leader's disk for performance and not for space, it is not necessary to scale the read-only replicas.

- Replication is asynchronous, and might be subject to replication lag. By default, there is no communication between the primary and replica regarding consistency. It is possible for a read-only replica to fall far enough behind that it needs to be resynced. Replication lag can be greater when the replica is in a region far away geographically from its leader.

- A read-only replica is a deployment with single data member and does not have any internal high-availability. It is prone to temporary interruptions and downtime during maintenance. If you have applications that rely on read-only replicas, be sure to have logic to retry failed queries, or load-balancing over multiple read-only replicas.

## The Leader

On the _Settings_ tab of a {{site.data.keyword.databases-for-postgresql}} deployment, before any read-only replicas have been provisioned, the _Replication_ panel has a **Create Read Replica** button.

![Replication panel before provisioning a replica](images/replica-before.png)

If a deployment is a leader and has a read-only replica that is already attached to it, then the _Replication_ panel has a list of replica deployments and a link to each one. Click on the cog to the right of the read-only replica's deployment name to manage it.

![List of replicas attached to a leader](images/replica-after.png)

## Provisioning a Read-only Replica

You can provision a read-only replica from the leader's _Settings_ panel by clicking **Create Read-Only Replica**. The source instance is automatically filled in. The read-only replica's name is auto-generated in the _Service Name_ field, but you can rename it freely. You can choose the region to deploy it in, and its initial memory allocation. Disk size, version, and public/private endpoints are automatically configured to match the settings of the leader deployment.

If you use [Key Protect](/docs/services/databases-for-postgresql?topic=cloud-databases-key-protect), Bring Your Own Key (BYOK) is supported only when provisioning from the CLI and API. Otherwise, the read-only replica is encrypted with a generated key. 
{: .tip}

### Provisioning through the API or the CLI

Provisioning a read-only replica through the CLI and the API works similarly to [provisioning a standard {{site.data.keyword.databases-for-postgresql}} deployment](/docs/services/databases-for-postgresql?topic=cloud-databases-provisioning). Provisioning is handled by the Resource Controller, and it uses a parameter `{"remote_leader_id": "crn:v1:..."}` to specify the leader of the replica you are provisioning.

For example, to provision a read-only replica through the CLI,
```
ibmcloud resource service-instance-create <replica_name> databases-for-postgresql standard <region> \
-p \ '{
  "remote_leader_id": "crn:v1:bluemix:public:databases-for-postgresql:us-south:a/54e8ffe85dcedf470db5b5ee6ac4a8d8:1b8f53db-fc2d-4e24-8470-f82b15c71819::",
  "members_memory_allocation_mb": "2048",
  "members_disk_allocation_mb": "10240"
}'
```

The same parameter is used to provision a read-only replica through the Resource Controller API.
```
curl -X POST \
  https://resource-controller.bluemix.net/v2/resource_instances \
  -H 'Authorization: Bearer <>' \
  -H 'Content-Type: application/json' \
    -d '{
    "name": "<replica_name>",
    "target": "<region>",
    "resource_group": "<your_resource_group_id>",
    "resource_plan_id": "databases-for-postgresql-standard",
    "remote_leader_id": "crn:v1:bluemix:public:databases-for-postgresql:us-south:a/54e8ffe85dcedf470db5b5ee6ac4a8d8:1b8f53db-fc2d-4e24-8470-f82b15c71819::",
    "members_memory_allocation_mb": "2048",
    "members_disk_allocation_mb": "10240"
  }'
```

For both the CLI and API commands, you have to specify both the RAM and disk amounts, keeping in mind the minimum size is 2 GB RAM and 10 GB disk. You can optionally specify whether the read-only replica should use public or private endpoints. You are not able to specify a version for the read-only replica. The version is automatically set to the same major version as the leader deployment.

## The Read-only Replica

On the _Settings_ tab of a read-only replica, the _Replication_ panel contains its name and region, and the name and region of its leader. It also has buttons to resync the read-only replica and to promote it.

![Replication Panel of a read-only replica](images/replica-roreplica.png)

### Checking Replication Status

Replication status is not automatically monitored, you have to monitor replication.

You can check the replication status of a read-only replica with `psql`, but only from its leader. [Connect to the leader deployment with `psql`](/docs/services/databases-for-postgresql?topic=databases-for-postgresql-connecting-psql) using the [admin credentials](/docs/services/databases-for-postgresql?topic=databases-for-postgresql-user-management#the-admin-user). Once you are connected run either
- for PostgreSQL version 10 and above `SELECT * from pg_stat_replication;`
Or
- for PostgreSQL version 9.x and below `SELECT * FROM get_pg_stat_replication();`

### Read-only Replica Users

Any user on the leader can log into and execute reads on a read-only replica with the same privileges that they have on the leader. You can also create users with access to the read-only replica from the read-only replica.

When you create a user through the read-only replica, it gets first created on the leader and then propagated out to read-only replica. If you have more than one read-only replica that is attached to a leader, a user that is created on any one of the read-only replicas is also created on all of the other read-only replicas. Read-replica users and their credentials can be made through [_Service Credentials_](/docs/services/databases-for-postgresql?topic=databases-for-postgresql-connection-strings#generating-connection-strings-from-service-credentials), the [CLI](/docs/services/databases-for-postgresql?topic=databases-for-postgresql-connection-strings#generating-connection-strings-from-the-command-line), and the [API](/docs/services/databases-for-postgresql?topic=databases-for-postgresql-connection-strings#generating-connection-strings-from-the-api).

Read-only replica users are able connect to the replicas and execute reads. Read-only replica users are not able to connect and execute operations on the leader.

Read-only replica users are assigned privileges by the leader, and are assigned the `ibm-cloud-base-user-ro` role, and are members of the `ibm-cloud-base-user` group. They have access to all of the objects that are created by other members of this group, including any users on the leader that were created through _Service Credentials_, the CLI, or the API. Consistent with privileges of the `ibm-cloud-base-user`, the read-only replica user does not have access to objects created by the admin user, or other users created through `psql`. For more information, see the [PostgreSQL Roles and Privileges](/docs/services/databases-for-postgresq?topic=databases-for-postgresql-user-management) page.

Write operations on the read-only replica for all users are not filtered or rejected, but fail at DB level.
{: .tip}


## Resyncing a Read-only Replica

If you need to resync a read-only replica, click the **Resync Read-Only Replica** button. Resyncing is a disruptive operation and performing a resync tears down and rebuilds the data in the read-only replica. The read-only replica is not able to perform any other operations or run any queries while a resync is running. Queries are not rerouted to the leader, so any connections to the read-only replica fail until it is finished resyncing. 

The amount of time it takes to resync a read-only replica varies, but the process can be very long running.
{: .tip}

To kick off a resync through the CLI, use the [cdb read-replica-resync](/docs/databases-cli-plugin?topic=cloud-databases-cli-cdb-reference#read-replica-resync) command.
```
ibmcloud cdb read-replica-resync <deployment name>
```

To kick off a resync through the API, send a POST to the [/deployments/{id}/remotes/resync](https://cloud.ibm.com/apidocs/cloud-databases-api#resync-read-only-replica) endpoint.
```
curl -X POST \
  https://api.{region}.databases.cloud.ibm.com/v4/ibm/deployments/{id}/remotes/resync \
  -H 'Authorization: Bearer <>' 
```

## Promoting a Read-only Replica

A read-only replica is able to be promoted to an independent cluster that can accept write operations as well as read operations. If something happens to the leader deployment, the read-only replica can be promoted to a stand-alone cluster and start accepting writes from your application. 

To promote a read-only replica from the UI, click the **Promote Read-Only Replica** button.

Upon promotion, the read-only replica terminates its connection to the leader and becomes a stand-alone {{site.data.keyword.databases-for-postgresql}} deployment. The deployment can start accepting and executing read and write operations, backups are enabled, and it is issued its own admin user. A new data member is added so the deployment becomes a cluster with two data members. This increases the cost as it is billed at the same per member consumption rate, but the deployment has two members instead of one.

When promoting a read-only replica, you have the option to skip the initial backup that would normally be taken upon promotion. Skipping the initial backup means that your replica becomes available more quickly, but there is no immediate backup available. You can start an on-demand backup once the promotion process is complete.

Once a read-only replica is promoted to an independent deployment, it is not possible to revert it back to a read-only replica or have it rejoin a leader.

To promote through the CLI, use the [cdb read-replica-promote](/docs/databases-cli-plugin?topic=cloud-databases-cli-cdb-reference#read-replica-promote) command.
```
ibmcloud cdb read-replica-promote <deployment name>
```

To promote through the API, send a PATCH to the [/deployments/{id}/remotes](https://cloud.ibm.com/apidocs/cloud-databases-api#modify-read-only-replication-on-a-deployment) endpoint. Setting the leader to the empty string is what initializes the promotion.
```
curl -X PATCH \
  https://api.{region}.databases.cloud.ibm.com/v4/ibm/deployments/{id}/remotes \
  -H 'Authorization: Bearer <>'  \
 -H 'Content-Type: application/json' \
 -d '{"remotes": {"leader": ""}}' \ 
```
