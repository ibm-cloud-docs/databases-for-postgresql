---
copyright:
  years: 2017,2018
lastupdated: "2017-07-18"
---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}

# Connecting an external application
{: #connecting-external-app}

{{site.data.keyword.databases-for-postgresql_full}} provides connection string information for you to connect your applications, `psql`, and other third-party PostgreSQL applications. 

## Generating Connection Strings via _Service Credentials_

To generate and view connection information all in one place, use the _Service Cedentials_ panel from the left-sidebar. Click on **New Credential** and give it a descriptive name. Click **Add** to provision the new credentials. This method will auto-generate a username and password, and make an associated database user in the PostgreSQL database.

### Using Service IDs

Since {{site.data.keyword.databases-for-postgresql}} is an IAM service, you can use [Service IDs](https://console.{DomainName}/docs/iam/serviceid.html#serviceids) to manage access to this service. For example, using a service ID that has an API key associated with it will grant that API key to access the {{site.data.keyword.cloud}} Databases API to administrate this service. If you have a Service ID, enter it's information under _Select Service ID_. 

### Generating _Service Credentials_ for existing users.

If you have an exisiting PostgreSQL user, created through the API, the IBM Cloud CLI, or through `psql`, and you would like to generate service credentials for them, enter the user name and password in the JSON field below _Add Inline Configuration Parameters_, or specify a file where the JSON information is being stored. For example, `{"existing_credentials":{"username":"Robert","password":"supersecure"}}`. This does not check for or create an associated PostgreSQL user.

## Generating Connection Strings via API

You may also generate and view connection strings through the API. To use the API to provision a new credential, send a `POST` request to the `https://api.{region}.databases.cloud.ibm.com/v4/{platform}/deployments/{id}/users` endpoint. Send in the desired username and password in the body of the request.  
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

## Using Connection Information

If you use the API or the _Service Credentials_ panel, connection information is returned in a JSON object in the "postgres" field. The below table describes the sub-fields of connection information.

Field Name|Description
----------|-----------
`authentication`|Basic authentication information: username, password, and auth type.
`certificate`|A self-signed certificate that is used to confirm that an application is connecting to the appropriate server. This is base64 encoded. You need to decode the key before using it.
`composed`|The URI to be used when connecting to the service. Includes the schema (`postgres:`), admin user name and password, host name of server, port number to connect to, database name and "?sslmode=verify-full" to enable SSL connections and verify the server.
`database`|The default database created at deployment provision.
`hosts`|The hostname and port of the deployment.
`path`|The path to the default database on the deployment.
`query_options`|The optional arguements included in the `composed` connection string.
`scheme`|The type of database that is offered by the service; in this case `postgresql`.
`type`|The format of the `composed` connection string.
{: caption="Table 1. PostgreSQL connection information" caption-side="top"}

## Using the self-signed certificate

Embedded in the connection information is the self-signed certificate that you can use in your applications to verify the server when you connect to it. Drivers for the various langagues will have a variety of TLS/SSL methods to achieve this, and you will have to consult the driver's documentation for specifics. As a broad overview, the steps are to:
- Download and save a copy of the certificate locally
- Decode it from base64 into a .pem certificate
- Provide the certifcate's path to the driver, and set the SSL mode to "Verify"


 
