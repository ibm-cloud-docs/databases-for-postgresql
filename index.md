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

# About {{site.data.keyword.databases-for-postgresql_full_notm}}
{: #about-databases-for-postgresql}


{{site.data.keyword.databases-for-postgresql}} is a managed PostgreSQL service hosted in the {{site.data.keyword.cloud_notm}} and integrated with other {{site.data.keyword.cloud_notm}} services. It is alo currently highly-experimental and should not be used for production workloads in its current state.

## Creating a {{site.data.keyword.databases-for-postgresql}} service

You can create a {{site.data.keyword.databases-for-postgresql}} service from the {{site.data.keyword.cloud_notm}} catalog.

Choose a service name, and a region, organization and space to provision the service in. You can use the _Select a database version_ field to choose from the available database versions and the _Select an initial memory allocation_ to set the initial amount of RAM for your service. The provision process will be started and take you back to your {{site.data.keyword.cloud_notm}} dashboard.
 
### Creating a service via the {{site.data.keyword.cloud_notm}} CLI

You can create a service through the {{site.data.keyword.cloud_notm}} CLI using the `service create` command.
```
ibmcloud service create databases-for-postgresql standard SERVICE_INSTANCE_NAME
```
`SERVICE_INSTANCE_NAME` is the name for your new service instance.

### Setting the admin password

Once the provisioning process is complete, set the password for the administrative user on your deployment. To set the password through the {{site.data.keyword.cloud_notm}} dashboard, select _Manage_ from the left sidebar and open the management panel for your service. Open the _Settings_ tab, and use the _Change Password_ panel to set a new admin password.

You can also set the admin user password using the API. Send a `PATCH` request to the  `https://api.{region}.databases.cloud.ibm.com/v4/ibm/deployments/{id}/users/{username}` endpoint. Use your deployment's id (CRN) and admin for the username. Specify the new password in the body of the request. An example is in the [API Reference](https://pages.github.ibm.com/compose/apidocs/apiv4doc-static.html#operation/changeUserPassword)

## Managing {{site.data.keyword.databases-for-postgresql}}

{{site.data.keyword.databases-for-postgresql}} is an IAM integrated service. Access to the service is governed by the roles and attributes that are consistent with IAM integrated services across the {{site.data.keyword.cloud_notm}}. Get started with managing your users on the [IAM Getting Started tutorial](https://console.{{DomainName}}/docs/iam/quickstart.html#getstarted). For more information on IAM, see the [What is IAM?](https://console.{{DomainName}}/docs/iam/index.html#iamoverview) documentation.

There are a few ways to manage your {{site.data.keyword.databases-for-postgresql}} service:

- You can manage your service by selecting _Manage_ from the left sidebar and opening the management panel from your service. Here you can find information about your {{site.data.keyword.databases-for-postgresql}} database. Administative settings are availble in the [Settings](./dashboard-settings.html) tab and backups are available through the [Backups](./dashboard-backups.html) tab.

- You can use the {{site.data.keyword.cloud_notm}} Databases API to manage your service. Authentication is IAM-based, so use your {{site.data.keyword.cloud_notm}} account's platform API keys when accessing the API. More information on API keys is in the [IAM documentation](https://console.{{DomainName}}/docs/iam/apikeys.html#platform-api-keys). The API is available at `https://api.{{region}}.databases.cloud.ibm.com/v4/ibm/` and the full API reference is available on [github](https://pages.github.ibm.com/compose/apidocs/apiv4doc-static.html).

- You can manage your service through the {{site.data.keyword.cloud_notm}} CLI. If you haven't already downloaded and installed it, get it [here](https://console.{{DomainName}}/docs/cli/index.html#overview). Once you have the {{site.data.keyword.cloud_notm}} CLI, there is an {{site.data.keyword.cloud_notm}} Databases plugin available. Download the latest release from it's [release page](https://github.ibm.com/compose/ibmcloud-dbs-plugin/releases), unzip it, and then install using `ibmcloud plugin install ibmcloud-dbs-plugin -f`. Once you have it installed, run `ibmcloud dbs help` for other commands and usage information. You can bring up the PostgreSQL admin user's connection strings with `ibmcloud dbs connections "your_service_name"`.

## Connecting an application to {{site.data.keyword.databases-for-postgresql}}

Connection strings and connection information to connect applications is available through _Service Credentials_ in the left sidebar. Find more information on [Connecting an External Application](./connecting-external.html).




