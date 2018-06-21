---

Copyright:
  years: 2018
lastupdated: "2018-06-19"
---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}

# About ICD for PostgreSQL

ICD for PostgreSQL is a managed PostgreSQL service hosted in the IBM Cloud and integrated with other IBM Cloud services. It feature high-availablilty, replication, and backups. It is alo currently highly-experimental and should not be used for production workloads in its current state.

## Creating

You can create a ICD for PostgreSQL service from the ICD page in the {{site.data.keyword.cloud_notm}} catalog.

Choose a service name, and a region, organization and space to provision the service in. You can use the Select a database version field to choose from the available database versions.

When you provision your {{site.data.keyword.ICD-postgres}} instance you can choose the Public Compute or Dedicated Compute plans. With the Dedicated plan, you can provision your {{site.data.keyword.ICD-Postgres}} instance into an available Dedicated cluster, which provides the security and isolation that is required by enterprise compliance and uses dedicated networking to ensure the performance of the deployed databases. 

### Creating a service via the {{site.data.keyword.cloud_notm}} CLI

Once you are logged in through the CLI, use
`ibmcloud service create databases-for-postgresql PLAN SERVICE_INSTANCE_NAME`

Where the `SERVICE_INSTANCE_NAME` is the name you would like to give to the service. `PLAN` is either "Public Compute" or "Dedicated Compute"


## Manage

You can manage your service from the service dashboard. Here you can find information about your ICD Postgresql database. You can:

  - manage your backups
  - allocate more resources for your service

For more information, see [Settings]().

## Connecting to {{ICD}}

Connection strings and connection infomation is available through _Service Credentials_ in the left sidebar. You can connect to your service by using the credentials that are created along with the service, or with your own created users and credentials. 

For more information the [Connecting to {{ICD-postgresql}}]().

{{site.data.keyword.ICD Postgres}} is an IAM integrated service. Access to the service is governed by the roles and attributes that are consistent with IAM integrated services across the {{site.data.keyword.cloud_notm}}. Get started with managing your users on the [IAM Getting Started tutorial](). For more information on IAM, see the [What is IAM?]() documentation.


