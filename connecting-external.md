---

Copyright:
  years: 2018
lastupdated: "2018-06-25"
---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}
{:tip: .tip}

# Connecting an external application
{: #connecting-external-app}

Connection information is available in the _Service Credentials_ section from the left sidebar. In order to connect an application or service to {{site.data.keyword.databases-for-postgresql}}, you will need to make at least one service credential. Click on **New Credential**, give a name to the credential, and click **Add** to provision the new credential. Both the username and password are auto-generated, and filled into the connection strings provided under _View Credentials_. The below table describes the types of connections to your service that are available.

Field Name | Description
----------|-----------
postgres | The URI to be used when connecting applications and drivers to the service. Includes the schema (postgres:), a user name and password, host name of server, port number to connect to, the self-sigend certificate in base64 format, and the default database name.
cli | A formatted `psql` shell command line that connects to the database instance, the self-signed certificate in base64 format, and the individual arguements to pass to `psql` when connecting.
instance_administration_api | API information specific to this service.
{: caption="Table 1. {{site.data.keyword.databases-for-postgresql}} credentials" caption-side="top"}
--------

## Generating Credentials via the API

To use the API to provision a new credential, send a `POST` request to the `https://api.{region}.databases.cloud.ibm.com/v4/{platform}/deployments/{id}/users` endpoint. Send in the desired username and password in the body of the request.

## Using self-signed certificates

How you pass the certificate information to your applications will depend on the drivers and libraries you are using. You may need to save a local copy of the certificate and provide its path to the driver. Or you may need to add the certificate to a certificate store. 

