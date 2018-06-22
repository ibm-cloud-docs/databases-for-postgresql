---

Copyright:
  years: 2018
lastupdated: "2018-05-11"
---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}
{:tip: .tip}

# Connecting to {{site.data.keyword.databases-for-postgresql_full-notm}}

Connection information is available in the _Service credentials_ section from the left sidebar. The below table describes the types of connections that are available to your service.

Field Name | Description
----------|-----------
postgres | The URI to be used when connecting to the service. Includes the schema (postgres:), admin user name and password, host name of server, port number to connect to, and database name.
cli | A `psql` shell command line that connects to the database instance.
instance_administration_api | API information specific to this service.
{: caption="Table 1. ICD for PostgreSQL credentials" caption-side="top"}
--------

## Managing Service Credentials

The default user for a {{site.data.keyword.databases-for-postgresql}} service comes with an 'admin' user with credentials. To view the 'admin' credentials and connection strings, click on the **New credential** button. You can name the credential, but otherwise leave the fields blank.

If you have an exisiting PostgreSQL user, and you would like to generate service credentials for them, enter their user name and password in the JSON field below _Add Inline Configuration Parameters_. An example, `{"existing_credentials":{"username":"Robert","password":"supersecure"}}`

Creating service credentials does not create an associated PostgreSQL user, nor does it check to see if an associated user already exists.
{: .tip}

## `psql`


## API

The {{site.data.keyword.cloud_notm}} Databases API is supplemental layer which adds the capability to manage {{site.data.keyword.cloud_notm}} Databases through a REST API. 

## `ibmcloud dbs` CLI Plugin

You can connect to your {{site.data.keyword.databases-for-postgresql}} through the {{site.data.keyword.cloud_notm}} CLI. If you haven't already downloaded and installed it, get it [here](https://console.bluemix.net/docs/cli/index.html#overview).

Once you have the {{site.data.keyword.cloud_notm}} CLI, there is an {{site.data.keyword.cloud_notm}} Databases plugin available. Download the latest release from it's [release page](https://github.ibm.com/compose/ibmcloud-dbs-plugin/releases), unzip it, and then install using 
`ibmcloud plugin install ibmcloud-dbs-plugin -f`

Once you have it installed and logged in with `ibmcloud login`, run `ibmcloud dbs` to see usage information.
