---

copyright:
  years: 2020, 2022
lastupdated: "2022-08-10"

keywords: troubleshooting for PostgreSQL, postgresql max_connections, postgres max connections, postgresql connection pooling, postgres connection pooling

subcollection: databases-for-postgresql

content-type: troubleshoot

---

{:tsSymptoms: .tsSymptoms}
{:tsCauses: .tsCauses}
{:tsResolve: .tsResolve}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:support: data-reuse='support'}
{:codeblock: .codeblock}
{:pre: .pre}
{:note:.deprecated}
{:troubleshoot: data-hd-content-type='troubleshoot'}
 

# How do I increase the max_connections on my {{site.data.keyword.databases-for-postgresql_full}} deployment?
{: #troubleshoot-max-connect}
{: troubleshoot}
{: support}

If you encounter an error when attempting to increase the `max_connections` on your {{site.data.keyword.databases-for-postgresql_full}} deployment, review these solutions.
{: shortdesc}

You receive a `superuser` error message when attempting the increase the `max_connections` on your {{site.data.keyword.databases-for-postgresql_full}} deployment.
{: tsSymptoms}

Review the following information to troubleshoot and resolve your `max_connections` issues:
{: tsResolve}

* The preferred method for this process is to use [Connection Pooling](/docs/databases-for-postgresql?topic=databases-for-postgresql-managing-connections#connection-pooling). Connection pooling prevents you from exceeding the connection limit and ensures that connections from your applications are being handled efficiently.{: .note}
* Alternatively, see [Changing your {{site.data.keyword.databases-for-postgresql_full}} Configuration](/docs/databases-for-postgresql?topic=databases-for-postgresql-changing-configuration).
