---

Copyright:
  years: 2018
lastupdated: "2018-07-01"
---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}

# About {{site.data.keyword.databases-for-postgresql_full_notm}}
{: #about-databases-for-postgresql}


{{site.data.keyword.databases-for-postgresql}} is a managed PostgreSQL service hosted in the {{site.data.keyword.cloud_notm}} and integrated with other {{site.data.keyword.cloud_notm}} services. It is alo currently highly-experimental and should not be used for production workloads in its current state.

## Creating a {{site.data.keyword.databases-for-postgresql}} service
{: #creating-databases-for-postgresql-service}

You can create a {{site.data.keyword.databases-for-postgresql}} service from the {{site.data.keyword.cloud_notm}} catalog.

Choose a service name, and a region, organization and space to provision the service in. You can use the _Select a database version_ field to choose from the available database versions and the _Select an initial memory allocation_ to set the initial amount of RAM for your service. The provision process will be started and take you back to your {{site.data.keyword.cloud_notm}} dashboard.
 
### Creating a service via the {{site.data.keyword.cloud_notm}} CLI

You can create a service through the {{site.data.keyword.cloud_notm}} CLI using the `service create` command.
```
ibmcloud resource service-instance-create SERVICE_INSTANCE_NAME databases-for-postgresql standard REGION
```
SERVICE_INSTANCE_NAME is the name for your new service instance, and REGION is where you want the service to be located.

## Managing {{site.data.keyword.databases-for-postgresql}}

{{site.data.keyword.databases-for-postgresql}} is an IAM integrated service. Access to the service is governed by the roles and attributes that are consistent with IAM integrated services across the {{site.data.keyword.cloud_notm}}. Get started with managing your users on the [IAM Getting Started tutorial](https://console.{DomainName}/docs/iam/quickstart.html#getstarted). For more information on IAM, see the [What is IAM?](https://console.{DomainName}/docs/iam/index.html#iamoverview) documentation.

For the reference on IAM roles and actions for {{site.data.keyword.databases-for-postgresql}} service instances is available on the [Access Management](./access-management.html) page.

Once you or another users have access to the service, there are a few ways to manage it.

- You can manage your service by selecting _Manage_ from the left sidebar and opening the management panel from your service. Here you can find information about your {{site.data.keyword.databases-for-postgresql}} database. Administative settings are availble in the [Settings](./dashboard-settings.html) tab and backups are available through the [Backups](./dashboard-backups.html) tab.

- You can use the {{site.data.keyword.cloud_notm}} Databases API to manage your service. Authentication is IAM-based, so use your {{site.data.keyword.cloud_notm}} account's platform API keys when accessing the API. More information on API keys is in the [IAM documentation](https://console.{DomainName}/docs/iam/apikeys.html#platform-api-keys). The API is available at the Foundataion Endpoint displayed on the _Overview_ page when you open the _Mange_ panel of your service. The full API reference is available on [github](https://pages.github.ibm.com/compose/apidocs/apiv4doc-static.html).

- You can manage your service through the {{site.data.keyword.cloud_notm}} CLI. If you haven't already downloaded and installed it, get it [here](https://console.{DomainName}/docs/cli/index.html#overview). Once you have the {{site.data.keyword.cloud_notm}} CLI, there is an {{site.data.keyword.cloud_notm}} Databases plugin available. Download the latest release from its [release page](https://github.ibm.com/compose/ibmcloud-dbs-plugin/releases), unzip it, and then install using `ibmcloud plugin install ibmcloud-dbs-plugin -f`. Once you have it installed, run `ibmcloud dbs help` for other commands and usage information. 

## Connecting to {{site.data.keyword.databases-for-postgresql}}

You can connect to your deployement using the connection strings and command-line information that are provided upon provision of your service.

## Connecting an {{site.data.keyword.cloud_notm}} application to {{site.data.keyword.databases-for-postgresql}}

To connect an {{site.data.keyword.cloud_notm}} application to your service, use credentials that are created in the _Service Credentials_ panel. You can find information on how to connect an {{site.data.keyword.cloud_notm}} application to a {{site.data.keyword.databases-for-postgresql}} service in [Connecting an {{site.data.keyword.cloud_notm}} Application](./connecting-ibmcloud-app.html).

## Connecting to {{site.data.keyword.databases-for-postgresql}} from outside {{site.data.keyword.cloud_notm}}

If you want to connect to {{site.data.keyword.databases-for-postgresql}} from outside {{site.data.keyword.cloud_notm}}, you can use the provided connection strings or command-line. You can find information on how to connect in [Connecting an external application](./connecting-external.html).


