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

## Generating Connection Strings from _Service Credentials_

1. Navigate to the service dashboard for your service.
2. Click _Service Credentials_ to open the _Service Credentials_ panel.
3. Click **New Credential**.
4. Choose a descriptive name for your new credential. 
5. Click **Add** to provision the new credentials. A username and password, and an associated database user in the PostgreSQL database are auto-generated.

### Using Service IDs

Because {{site.data.keyword.databases-for-postgresql}} is an IAM service, you can use [Service IDs](https://console.{DomainName}/docs/iam/serviceid.html#serviceids) to manage access to this service. For example, using a service ID that has an API key associated with it grants that API key access to the {{site.data.keyword.cloud_notm}} Databases API to administer this service. If you have a Service ID, enter its information under _Select Service ID_. 

### Generating _Service Credentials_ for existing users.

You can generate service credentials for an existing PostgreSQL user that was created through the API, the IBM Cloud CLI, or by using `psql`.

Enter the user name and password in the JSON field below _Add Inline Configuration Parameters_, or specify a file where the JSON information is stored. For example, `{"existing_credentials":{"username":"Robert","password":"supersecure"}}`.

This does not check for or create an associated PostgreSQL user.
{: tip}

## Generating Connection Strings via API

To use the API to provision a new credential, send a `POST` request to the `https://api.{region}.databases.cloud.ibm.com/v4/{platform}/deployments/{id}/users` endpoint. Send the desired username and password in the body of the request.

The following code creates a user "mary" with password "mostsecure".

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

When the credentials have been created, you can view their associated connection information by sending a `GET` request to the `https://api.{region}.databases.cloud.ibm.com/v4/ibm/deployments/{id}/users/{userid}/connections` endpoint. 

For more information, see the [API Reference](https://pages.github.ibm.com/compose/apidocs/apiv4doc-static.html#operation/createDatabaseUser).

## Using Connection Information

If you use the API or the _Service Credentials_ panel, connection information is returned in a JSON object in the "postgres" field. The table describes the sub-fields of connection information.

Field Name|Description
----------|-----------
`authentication`|Basic authentication information: username, password, and auth type.
`certificate`|A self-signed certificate that is used to confirm that an application is connecting to the appropriate server. This is base64 encoded. You need to decode the key before using it.
`composed`|The URI to be used when connecting to the service. Includes the schema (`postgres:`), admin user name and password, host name of server, port number to connect to, database name and "?sslmode=verify-full" to enable SSL connections and verify the server.
`database`|The default database created at deployment provision.
`hosts`|The hostname and port of the deployment.
`path`|The path to the default database on the deployment.
`query_options`|The optional arguments included in the `composed` connection string.
`scheme`|The type of database that is offered by the service; in this case `postgresql`.
`type`|The format of the `composed` connection string.
{: caption="Table 1. PostgreSQL connection information" caption-side="top"}

## Using the self-signed certificate

The connection information includes a self-signed certificate that you can use in your applications to verify the server when you connect to it. Different drivers for the various languages use a variety of TLS/SSL methods: check the documentation for the driver you are using for specific information on how it handles TLS/SSL.

Usually, you need to complete the following steps.

1. Download and save a copy of the certificate locally
2. Decode it from base64 into a .pem certificate
3. Provide the certificate's path to the driver, and set the SSL mode to "Verify"


 
