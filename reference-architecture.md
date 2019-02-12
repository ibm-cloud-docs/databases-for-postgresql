---

Copyright:
  years: 2018, 2019
lastupdated: "2019-01-14"
---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}

# Architecture and High-Availability

{{site.data.keyword.databases-for-postgresql_full}} is a managed cloud database service that runs in containers that are orchestrated by Kubernetes. It is fully integrated into the {{site.data.keyword.cloud_notm}} ecosystem. The database, storage, and monitoring all run in {{site.data.keyword.cloud_notm}}.

## PostgreSQL Databases

A {{site.data.keyword.databases-for-postgresql}} service contains a cluster with two data members, a leader and a replica. Data is replicated to both data members. When you provision a {{site.data.keyword.databases-for-postgresql}} deployment in a particular region, your database members are distributed among the [data centers](https://{DomainName}/docs/overview/zero_downtime.html#data_center) available in that region. This replicates your data geographically within the region, so if one data center goes down, your database remains available Currently, no cross-region replication available.

High availability is built on top of asynchronous replication, with a distributed consensus mechanism to maintain global cluster state and handle failovers. If one database member becomes unreachable, your cluster continues to operate normally.

### Storage

Database storage for {{site.data.keyword.databases-for-postgresql}} is provided on {{site.data.keyword.cloud_notm}} [Block Storage](https://{DomainName}/docs/infrastructure/BlockStorage/index.html#getting-started-with-block-storage). Backup storage is on {{site.data.keyword.cloud_notm}} [Object Storage](https://{DomainName}/docs/services/cloud-object-storage/about-cos.html#about-ibm-cloud-object-storage).

### Full-disk Encryption

All {{site.data.keyword.databases-for-postgresql}} services all have encryption at rest. Both your data and your backups reside on servers that have volume-level encryption enabled. If your environment requires that you control the encryption keys, [Key Protect integration](./reference-key-protect.html) is available.

### Connection Limits 

{{site.data.keyword.databases-for-postgresql}} sets the maximum number of connections to your PostgreSQL database to **100**. If you see a string of connection failures and the error,
```
FATAL: remaining connection slots are reserved for
non-replication superuser connections
```
your application is exceeding this connection limit. The recommended solution is to make use of [connection pooling](#connection-pooling). It is possible to raise the connection limit, but connections to the database consume resources. To raise the connection limit you might have to [scale your deployment](./dashboard-settings.html#scaling-resources) and then you must [open a support ticket](https://cloud.ibm.com/unifiedsupport/cases/add). 

## Portals

{{site.data.keyword.databases-for-postgresql}} database connections are managed by 2 HAProxy portals which automatically connect to the leader member of the PostgreSQL cluster. The portals are then behind a Kubernetes endpoint, which provides the connection for your applications. Having two portals allows for applications to maintain connectivity if one of the portals becomes unreachable.

### Encryption in Transit

All {{site.data.keyword.databases-for-postgresql}} connections are TLS/SSL enabled and support TLS version 1.2. [Self-signed certificates](./connecting-external.html#driver-tls-and-self-signed-certificate-support) are provided to enable applications to validate the server upon connection.

## Application-level High-Availability

{{site.data.keyword.databases-for-postgresql}} provides replication, fail-over, and high-availability features to protect your databases and data from infrastructure maintenance, upgrades, and failures. By contrast, application resilience and connection error handling are the responsibility of the application designer. 

### Updates, Maintenance, and Network Interruptions

Applications that communicate over networks and cloud services are subject to transient connection failures. You want to design your applications to retry connections when errors are caused by a temporary loss in connectivity to your deployment or to {{site.data.keyword.cloud_notm}}.

Because {{site.data.keyword.databases-for-postgresql}} is a managed service, regular updates and database maintenance occurs as part of normal operations. This can occasionally cause short intervals where your database is unavailable. It may also cause a the database to trigger a graceful fail-over, retry, and reconnect. It takes a short time for the database to determine which member is a replica and which is the leader, so you might also see a short connection interruption. Failovers generally take less than 30 seconds.

Your applications have to be designed to handle temporary interruptions to the database, implement error handling for failed database commands, and implement retry logic to recover from a temporary interruption.

Several minutes of database unavailability or connection interruption is not expected and you should open a [support ticket](https://cloud.ibm.com/unifiedsupport/cases/add) with details if you have time periods longer than a minute with no connectivity so we can investigate.

### Resource Scaling

{{site.data.keyword.databases-for-postgresql}} does not auto-scale. Deployment owners should monitor the state of the deployment, estimate typical resource usage, and scale the deployment accordingly.

If you are planning on running operations that might put a spike in the usual RAM usage, or any data operations that could overflow your allotted storage, you should manually scale your service's resources up first to avoid hitting any resource limits that would affect deployment operations.

### Connection Pooling

Exceeding the [connection limit](#connection-limits) for your deployment can cause your database to be unreachable by your applications.

Application designers are responsible for monitoring and managing the number of connections that are being made to the database. Many PostgreSQL driver libraries have connection pooling classes and functions. You need to consult your driver's documentation to implement connection pooling that is optimal for your use case.

Alternatively, you can use a third-party tool such as [PgBouncer](https://pgbouncer.github.io/) to manage your application's connections.

## Access Management

{{site.data.keyword.databases-for-postgresql}} is an IAM-integrated service. Access to the service is governed by the roles and attributes that are consistent with IAM-integrated services across {{site.data.keyword.cloud_notm}}. For more information about IAM, see the [What is IAM?](https://{DomainName}/docs/iam/index.html#iamoverview) documentation. For more information on how IAM affects access to {{site.data.keyword.databases-for-postgresql}}, see [Managing Access](./access-management.html).

Database-level access can be managed through PostgreSQL's native [user management and database roles](https://www.postgresql.org/docs/current/static/database-roles.html).

## SLA

{{site.data.keyword.databases-for-postgresql}} conforms to the {{site.data.keyword.cloud_notm}} [SLA terms](https://cloud.ibm.com/docs/overview/zero_downtime.html#SLAs).

