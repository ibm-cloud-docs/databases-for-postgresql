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

ICD for PostgreSQL is TBC.

## Creating

Create things.

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





