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
{:tip: .tip}

# Administering your PostgreSQL databases

The {{site.data.keyword.databases-for-postgresql_full}} service is provisioned with an admin user, so you can administer PostgreSQL by using its command-line tool, `psql`.

The admin user comes with the PostgreSQL default role [`pg_monitor`](https://www.postgresql.org/docs/10/static/default-roles.html), allowing access to PostgreSQL monitoring views and functions. By default, the admin user does not have permissions on objects that are created by other users.

For security reasons, and as part of the infrastructure provided by {{site.data.keyword.databases-for-postgresql}} as a managed service, there is no superuser role available to the end-user.
{: .tip}

To get started, you will have to complete the following steps:

1. Set the admin credentials
2. Retrieve the admin connection strings and other connection information
3. Decode and save a copy of the self-signed certificate

## Setting the admin password

You have to set the admin password before you can use it to connect. To set the password through the {{site.data.keyword.cloud_notm}} dashboard, select _Manage_ from the service dashboard to open the management panel for your service. Open the _Settings_ tab, and use the _Change Password_ panel to set a new admin password.

You can also set the admin user password through the API. Send a `PATCH` request to the `https://api.{region}.databases.cloud.ibm.com/v4/ibm/deployments/{id}/users/admin` endpoint.

```
curl -X PATCH "https://api.{region}.databases.cloud.ibm.com/v4/ibm/deployments/{id}/users/admin" \
-H "Authorization: Bearer $APITOKEN" \
-H "Content-Type: application/json; charset=utf-8" \
-d \
  '{
  "user": 
    {
      "password":"lkdfj1ieo4fhhelk5aei2efjdsa"
    }
  }'
```

For more information, see the [API Reference](https://pages.github.ibm.com/compose/apidocs/apiv4doc-static.html#operation/changeUserPassword).

## Connecting with `psql`

To use `psql`, the PostgreSQL client tools need to be installed on the local system. They can be installed by installing the full PostgreSQL package downloaded from postgresql.org, or from your operating systems packages.

You can read more about psql in the PostgreSQL documentation - reference - and a simple introduction in Postgres Guide.

### Connecting to 'psql' with the CLI plug-in

The {{site.data.keyword.cloud_notm}} CLI cloud databases plug-in provides the admin user's connection string in URI format with the command: `ibmcloud dbs deployment-connections "your-service-name"`.

You can also connect to `psql` from the cloud databases plug-in with the admin user with `ibmcloud cdb deployment-connections "your-service-name" -u admin --start`. Enter the admin password when prompted.

### Getting a `psql` connection string with the API

To retrieve the admin user's connection strings through the API, send a GET request to `https://api.{region}.databases.cloud.ibm.com/v4/ibm/deployments/{id}/users/admin`. The JSON response includes the strings to connect to `psql` in "cli" field.

The `psql` connection information table describes the subfields of connection information.

Field Name|Description
----------|-----------
`arguments`|The information that is passed as arguments to the `psql` command,
`bin`|The package that this information is intended for; in this case `psql`.
`certificate`|A self-signed certificate that is used to confirm that an application is connecting to the appropriate server. It is base64 encoded. You need to decode the key before using it.
`composed`|A formatted `psql` command to establish a connection to your deployment.
`environment`|`psql` arguments that can be set and pulled from the environment.
`type`|The type of package that uses this connection information; in this case `cli`. 
{: caption="Table 1. `psql` connection information" caption-side="top"}

## Using the self-signed certificate

The formatted `psql` command sets the option to verify the server via certificate upon connection. To use this feature, you need to complete the following steps:

1. Download and save a copy of the certificate
2. Decode the certificate from base64 format to the .pem certificate format.
3. Provide the path to the certificate to the connection string in the `PGSSLROOTCERT` environment variable.

### CLI plug-in support for the self-signed certificate

You can display the decoded certificate for your deployment with the CLI plug-in with the `ibmcloud cdb deployment-cacert "your-service-name" command. Copy and save the command's output to a file and provide the file's path to the `PGSSLROOTCERT` environment variable.

## Users and Roles

If you need to [connect external services](./connecting-external.html), such as applications or third-party tools, use the cloud databases cli-plugin, the API, or  _Service Credentials_. Users created through use the cloud databases cli-plugin, the API, or  _Service Credentials_ have `Create role` and `Create DB` privileges. Those users also have ownership over the things that they create.

Users can also be created through `psql` using [PostgreSQL's native user management](https://www.postgresql.org/docs/10/static/user-manag.html). Please note that users created directly in PostgreSQL with `psql` are not governed by IAM or other access policies and mechanisms.

## The default `ibmclouddb` database

When the service is provisioned, it automatically creates a default database. There is nothing special about this database except that it is the database that is auto-filled into the connection strings provided with your service. You are free to use this database, or to create other databases and connect to them. If using other databases, be sure to fill the new database into the connection strings accordingly. If you do not wish to use the `ibmcloudb`, you can delete the database entirely.







