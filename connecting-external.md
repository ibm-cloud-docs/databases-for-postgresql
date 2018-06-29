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

Connection information is available in the _Service Credentials_ section from the left sidebar. In order to connect an application or service to {{site.data.keyword.databases-for-postgresql}}, you will need to make at least one service credential. Click on **New Credential**, give a name to the credential, and click **Add** to provision the new credential. Once you have created a set of service credetials for you application, you will find all of the connection information as JSON document under _Available Credentials_. It contains all the information that a driver or library will need to connect to your deployment. The below table describes the types of connections that are available.

Field Name | Description
----------|-----------
postgres | The URI to be used when connecting applications and drivers to the service. Includes the schema (postgres:), a user name and password, host name of server, port number to connect to, the self-sigend certificate in base64 format, and the default database name.
cli | A formatted `psql` shell command line that connects to the database instance, the self-signed certificate in base64 format, and the individual arguements to pass to `psql` when connecting.
instance_administration_api | API information specific to this service.
{: caption="Table 1. {{site.data.keyword.databases-for-postgresql}} credentials" caption-side="top"}
--------

## Generating Credentials via the API

To use the API to provision a new credential, send a `POST` request to the `https://api.{region}.databases.cloud.ibm.com/v4/{platform}/deployments/{id}/users` endpoint. Send in the desired username and password in the body of the request.  
For example, the `curl` to create user "Mary" with password "mostsecure":
```
curl -X POST "https://api.{region}.databases.cloud.ibm.com/v4/ibm/deployments/{id}/users" \
-H "Authorization: Bearer $APITOKEN" \
-H "Content-Type: application/json; charset=utf-8" \
-d \
  '{
  "user": 
    {
      "username":"Mary",
      "password":"mostsecure"
    }
  }'
```
More information can be found in the [API Reference](https://pages.github.ibm.com/compose/apidocs/apiv4doc-static.html#operation/createDatabaseUser)