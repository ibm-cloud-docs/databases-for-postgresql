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
{:tip: .tip}

# Connecting an {{site.data.keyword.cloud_notm}} application

To connect an {{site.data.keyword.cloud}} application to your service, use the connection information in the _Service Credentials_ section of your service. The below table describes the types of connections that are available.

Field Name | Description
----------|-----------
postgres | The URI to be used when connecting applications and drivers to the service. Includes the schema (postgres:), a user name and password, host name of server, port number to connect to, the self-sigend certificate in base64 format, and the default database name.
cli | A formatted `psql` shell command line that connects to the database instance, the self-signed certificate in base64 format, and the individual arguements to pass to `psql` when connecting.
instance_administration_api | API information specific to this service.
{: caption="Table 1. {{site.data.keyword.databases-for-postgresql}} credentials" caption-side="top"}
--------

## Generating a New Credential
To connect an {{site.data.keyword.cloud_notm}} application to {{site.data.keyword.databases-for-postgresql}}, you will need to make at least one service credential. Click on **New Credential** and give it a descriptive name. 

If you have a service already running in the {{site.data.keyword.cloud}}, can enter it's information under _Select Service ID_. If you just want new credentials, you can ignore this field or select _Auto Generate_. 

If you have an exisiting PostgreSQL user, and you would like to generate service credentials for, enter the user name and password in the JSON field below _Add Inline Configuration Parameters_, or specify a file where the JSON information is being stored. For example, `{"existing_credentials":{"username":"Robert","password":"supersecure"}}`. This does not check for or create an associated PostgreSQL user.

Click **Add** to provision the new Service Credential.

### Generating Credentials via the API

To use the API to provision a new credential, send a `POST` request to the `https://api.{region}.databases.cloud.ibm.com/v4/{platform}/deployments/{id}/users` endpoint. Send in the desired username and password in the body of the request.  
For example, the `curl` to create user "mary" with password "mostsecure":
```
curl -X POST "https://api.{region}.databases.cloud.ibm.com/v4/ibm/deployments/{id}/users" \
-H "Authorization: Bearer $APITOKEN" \
-H "Content-Type: application/json; charset=utf-8" \
-d \
  '{
  "user": 
    {
      "username":"mary",
      "password":"mostsecure"
    }
  }'
```

Once the credentials have been created, you can view their associated connection information by sending a `GET` request to the`https://api.{region}.databases.cloud.ibm.com/v4/ibm/deployments/{id}/users/{userid}/connections` endpoint. 

More information can be found in the [API Reference](https://pages.github.ibm.com/compose/apidocs/apiv4doc-static.html#operation/createDatabaseUser)
