---

Copyright:
  years: 2018
lastupdated: "2018-09-18"
---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}
{:tip: .tip}

# About {{site.data.keyword.databases-for-postgresql_full_notm}}
{: #about-databases-for-postgresql}

{{site.data.keyword.databases-for-postgresql_full}} is a managed PostgreSQL service that is hosted in the {{site.data.keyword.cloud_notm}} and integrated with other {{site.data.keyword.cloud_notm}} services. PostgreSQL provides a powerful, open source object-relational database that is highly customizable. With PostgreSQL, development is fast and easily scalable. You can develop in a language that you're comfortable with, such as C/C++, Perl, Python, TCL/TK, Delphi/Kylix, VB, PHP, ASP, and Java. You get a feature-rich enterprise database with JSON support, giving you the best of both the SQL and NoSQL worlds. 
{:shortdesc}

## Creating a {{site.data.keyword.databases-for-postgresql}} service
{: #creating-databases-for-postgresql-service}

You can create a {{site.data.keyword.databases-for-postgresql}} service from the {{site.data.keyword.cloud_notm}} catalog.

Create a service name and choose a region, organization, and space to provision the service in. You can use the _Select a database version_ field to choose from the available database versions and the _Select an initial memory allocation_ field to set the initial amount of RAM for your service. Click **Create** to start provisioning the service. You are taken back to your {{site.data.keyword.cloud_notm}} dashboard to monitor its progress.
 
### Creating a service via the {{site.data.keyword.cloud_notm}} CLI

You can create a service through the {{site.data.keyword.cloud_notm}} CLI using the `service create` command.
```
ibmcloud resource service-instance-create SERVICE_INSTANCE_NAME databases-for-postgresql standard REGION
```
SERVICE_INSTANCE_NAME is the name for your new service instance, and REGION is where you want the service to be located.

## {{site.data.keyword.databases-for-postgresql}} as an {{site.data.keyword.cloud_notm}} service

{{site.data.keyword.databases-for-postgresql}} provides a UI, accessible by selecting _Manage_ from the left sidebar of your service and opening the management panel. You get a quick [Overview](./dashboard-overview) of your service as well as configuration settings on the [Settings](./dashboard-settings.html) tab and access to your backups on the [Backups](./dashboard-backups.html) tab.

### Using the command line interface

The command line interface for {{site.data.keyword.databases-for-postgresql}} is provided by the {{site.data.keyword.cloud_notm}} CLI. If you need to download and install it, get it [here](https://console.{DomainName}/docs/cli/index.html#overview). Once you have the {{site.data.keyword.cloud_notm}} CLI, get the {{site.data.keyword.cloud_notm}} databases plugin. Download the latest release from its [release page](https://github.ibm.com/compose/ibmcloud-dbs-plugin/releases), unzip it, and then install using `ibmcloud plugin install ibmcloud-dbs-plugin -f`. Once you have it installed, run `ibmcloud cdb help` for other commands and usage information.

### Managing Access to {{site.data.keyword.databases-for-postgresql}}

{{site.data.keyword.databases-for-postgresql}} is an Identity and Access Management (IAM) integrated service. Access is governed by the roles and attributes that are consistent across IAM-integrated services in {{site.data.keyword.cloud_notm}}. Get started with managing your users on the [IAM Getting Started tutorial](https://console.{DomainName}/docs/iam/quickstart.html#getstarted). For more information on IAM, see the [What is IAM?](https://console.{DomainName}/docs/iam/index.html#iamoverview) documentation.

More information on IAM roles and actions for the {{site.data.keyword.databases-for-postgresql}} service is available on the [Access Management](./access-management.html) page.

### Using the cloud databases API

You can use the {{site.data.keyword.cloud_notm}} databases API to manage your service. Authentication is IAM-based and you use your {{site.data.keyword.cloud_notm}} account's platform API keys to access the API. More information on API keys is in the [IAM documentation](https://console.{DomainName}/docs/iam/apikeys.html#platform-api-keys). The API foundation endpoint for your service is on the _Overview_ page when you open the _Manage_ panel of your service. The full API reference is available on [GitHub](https://pages.github.ibm.com/compose/apidocs/).

## PostgreSQL Database Administration

Under the the {{site.data.keyword.databases-for-postgresql}} service, are the PostgreSQL databases. You can control and manage the databases directly. [Set the admin password](./admin-password.html) and [connect as the admin](./admin-connecting) using `psql`. Read more if you need to manage users and privileges or install database extensions.

## Connecting to {{site.data.keyword.databases-for-postgresql}}

General information on getting connection strings for your applications, for `psql`, and for users you provision on your account can be found on the [Getting Connection Strings](./using-connection-strings) page.

Specific guidance on connecting with PostgreSQL drivers is on the [Connecting External Applications](./connecting-external.html) page. If you want to connect a Cloud Foundry application running in {{site.data.keyword.cloud_notm}}, see the [Connecting an {{site.data.keyword.cloud_notm}} Application](./connecting-ibmcloud-app.html) page. The [Getting Started tutorial]() that provides a sample application that can run locally or on {{site.data.keyword.cloud_notm}} to test-drive your {{site.data.keyword.databases-for-postgresql}} deployment.

## Other {{site.data.keyword.cloud_notm}} Integrations

{{site.data.keyword.databases-for-postgresql}} offers other cloud services integrations. 
- Manage your own encryption keys with [Key Protect]()
- View events with [Activity Tracker]()









