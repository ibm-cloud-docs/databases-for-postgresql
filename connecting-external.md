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

Connection information is available in the _Service Credentials_ section from the left sidebar. The below table describes the types of connections to your service that are available.

Field Name | Description
----------|-----------
postgres | The URI to be used when connecting applications and drivers to the service. Includes the schema (postgres:), a user name and password, host name of server, port number to connect to, the self-sigend certificate in base64 format, and the default database name.
cli | A formatted `psql` shell command line that connects to the database instance, the self-signed certificate in base64 format, and the individual arguements to pass to `psql` when connecting.
instance_administration_api | API information specific to this service.
{: caption="Table 1. ICD for PostgreSQL credentials" caption-side="top"}
--------

## Using self-signed certificates

How you pass the certificate information to your applications will depend on the drivers and libraries you are using. You may need to save a local copy of the certificate and provide its path to the driver. Or you may need to add the certificate to a certificate store. 

