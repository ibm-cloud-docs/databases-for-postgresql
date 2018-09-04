---

Copyright:
  years: 2018
lastupdated: "2018-08-21"
---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}

# Architecture

{{site.data.keyword.cloud_notm}} Databases is a managed cloud database service that runs in containers that are orchestrated by Kubernetes. It is fully integrated into the {{site.data.keyword.cloud_notm}} ecosystem. The database, storage, and monitoring all run in {{site.data.keyword.cloud_notm}}.

## PostgreSQL Databases

An {{site.data.keyword.databases-for-postgresql_full}} service contains a cluster with two data members, a leader member and a follower member. Data is replicated across both data members, and their disk space is spread over the region's availability zones. High-availability is managed with [Patroni](https://github.com/zalando/patroni), where three etcd arbiters maintain cluster state, manage replication, and handle failover. If one data member becomes unreachable, your cluster continues to operate normally.

### Storage

All storage for {{site.data.keyword.cloud_notm}} Databases is provided on {{site.data.keyword.cloud_notm}} Object Storage. Storage for {{site.data.keyword.databases-for-postgresql}} is auto-scaled to the size of your data on disk.

### Full-disk Encryption

All {{site.data.keyword.databases-for-postgresql}} services all have encryption at rest. Your data resides on servers that have volume-level encryption enabled.

## Portals

{{site.data.keyword.databases-for-postgresql}} database connections are managed by 2 HAProxy portals. They automatically connect to the leader member of the PostgreSQL cluster and provide load-balancing. The portals are then behind the Kubernetes Nodeport Service, which provides the point of connection for your applications. Having two portals allows for applications to maintain connectivity if one of the portals becomes unreachable.

### Encryption in Transit

All {{site.data.keyword.databases-for-postgresql}} HAProxy portals are TLS/SSL enabled and support TLS version 1.2. Self-signed certificates are provided to enable applications to validate the server upon connection.

## Access Management

{{site.data.keyword.databases-for-postgresql}} is an IAM-integrated service. Access to the service is governed by the roles and attributes that are consistent with IAM-integrated services across the {{site.data.keyword.cloud_notm}}. For more information on IAM, see the [What is IAM?](https://console.{DomainName}/docs/iam/index.html#iamoverview) documentation. For more information on how IAM affects access to {{site.data.keyword.databases-for-postgresql}}, see [Managing Access](./access-management.html).

Database-level access can be managed through PostgreSQL's native [user management and database roles](https://www.postgresql.org/docs/current/static/database-roles.html). For more information on database-level administration in {{site.data.keyword.databases-for-postgresql}}, see the [Administering your PostgreSQL databases](./administering.html) page.

