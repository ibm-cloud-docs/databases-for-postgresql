---

Copyright:
  years: 2018, 2020
lastupdated: "2020-09-25"

keywords: postgresql, databases, connection limits

subcollection: databases-for-postgresql

---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}

# High-Availability
{: #high-availability}

{{site.data.keyword.databases-for-postgresql_full}} is a managed cloud database service that is fully integrated into the {{site.data.keyword.cloud_notm}} environment. The database, storage, and supporting infrastructure all run in {{site.data.keyword.cloud_notm}}.

{{site.data.keyword.databases-for-postgresql}} provides replication, fail-over, and high-availability features to protect your databases and data from infrastructure maintenance, upgrades, and failures. Deployments contain a cluster with two data members, a leader and a replica. Both members contain a copy of your data by using asynchronous replication, with a distributed consensus mechanism to maintain cluster state and handle failovers. If the leader becomes unreachable, the cluster initiates a failover and the replica is promoted to leader. The replica rejoins the cluster and your cluster continues to operate normally. 

{{site.data.keyword.databases-for-postgresql}} will, at times, do controlled switchovers under normal operation. These switchovers are no-data-loss events that result in resets of active connections. There is a period of up to 15 seconds where reconnections can fail. At times, unplanned failovers might occur due to unforeseen events on the operating environment. These can take up to 45 seconds, but can be less. In both cases, potential exists for the downtime to be longer.

You can extend high-availability to more regions and spread to more replicas by adding [read-only replicas](/docs/databases-for-postgresql?topic=databases-for-postgresql-read-only-replicass). 

## Application-level High-Availability

Applications that communicate over networks and cloud services are subject to transient connection failures. You want to design your applications to retry connections when errors are caused by a temporary loss in connectivity to your deployment or to {{site.data.keyword.cloud_notm}}.

Because {{site.data.keyword.databases-for-postgresql}} is a managed service, regular updates and database maintenance occurs as part of normal operations. This can occasionally cause short intervals where your database is unavailable. It can also cause the database to trigger a graceful fail-over, retry, and reconnect. It takes a short time for the database to determine which member is a replica and which is the leader, so you might also see a short connection interruption. Failovers generally take less than 30 seconds.

Your applications have to be designed to handle temporary interruptions to the database, implement error handling for failed database commands, and implement retry logic to recover from a temporary interruption.

Several minutes of database unavailability or connection interruption are not expected. Open a [support case](https://cloud.ibm.com/unifiedsupport/cases/add) with details if you have time periods longer than a minute with no connectivity so we can investigate.

## Connection Limits
{: #connection-limits-ha}

{{site.data.keyword.databases-for-postgresql}} sets the maximum number of connections to your PostgreSQL database to **115**. 15 connections are reserved for the superuser to maintain the state and integrity of your database, and 100 connections are available for you and your applications. After the connection limit has been reached, any attempts at starting a new connection results in an error. To prevent overwhelming your deployment with connections, use connection pooling, or scale your deployment and increase its connection limit. For more information, see the [Managing PostgreSQL Connections](/docs/databases-for-postgresql?topic=databases-for-postgresql-managing-connections) page.

## High availability, disaster recovery, and SLA resources

{{site.data.keyword.databases-for-postgresql}} deployments conform to the {{site.data.keyword.cloud_notm}} Databases [HA, DR, and SLA](/docs/cloud-databases?topic=cloud-databases-ha-dr) information and terms.

