---

Copyright:
  years: 2018
lastupdated: "2018-05-14"
---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}

# About ICD for PostgreSQL

ICD for PostgreSQL is a managed PostgreSQL service hosted in the IBM Cloud and integrated with other IBM Cloud services. It feature high-availablilty, replication, and backups. It is alo currently highly-experimental and should not  be used for production workloads in its current state.

## Creating

You can create a ICD for PostgreSQL service from the ICD page in the {{site.data.keyword.cloud_notm}} catalog.

Choose a service name, and a region, organization and space to provision the service in. You can use the Select a database version field to choose from the available database versions.

When you provision your {{site.data.keyword.composeForPostgreSQL}} instance you can choose the Public or Dedicated plans. 

### Creating a service via API

### Creating a service via the IBM Cloud CLI

## Manage

## Service Credentials

Connection information is available in the _Service credentials_ section from the left sidebar. The below table describes the types of connections that are available to your service.

Field Name	Description
postgres | The URI to be used when connecting to the service. Includes the schema (postgres:), admin user name and password, host name of server, port number to connect to, and database name.
cli | A `psql` shell command line that connects to the database instance.
instance_administration_api | API information specific to this service.
{: caption="Table 1. ICD for PostgreSQL credentials" caption-side="top"}
--------

### Managing Service Credentials

ICD for PostgreSQL is an IAM integrated service.

Default comes with 'admin' credentials. To view the 'admin' credentials and connection strings, click on the **New credential** button. You can name the credential, but otherwise leave the fields blank.

If you have an exisiting PostgreSQL user, and you would like to generate service credentials for them, enter their user name and password in the JSON field below _Add Inline Configuration Parameters_. An example, `{"existing_credentials":{"username":"Robert","password":"supersecure"}}`

Creating service credentials does not create an associated PostgreSQL user, nor does it check to see if an associated user already exists.

## Connections

Manage Cloud Foundry applications that are commected to this ICD for PostgreSQL service in the _Connections_ section. For more information on connecting an application that is running in the {{site.data.keyword.cloud}} see [Connecting an IBM Cloud Application](). An example application and tutorial is available in [Getting Started]()

## API Reference

## IBM Cloud CLI





