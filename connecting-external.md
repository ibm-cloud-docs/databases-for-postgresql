---
copyright:
  years: 2017,2018
lastupdated: "2017-07-01"
---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}

# Connecting an external application
{: #connecting-external-app}

{{site.data.keyword.databases-for-postgresql}_full}} provides all information to connect applications running anywhere, `psql`, and other third-party PostgreSQL applications. To connect to your deployment, use the admin user that is created when the service is provisioned. Once you have set up the admin credentials, retrieve the connection information associated with your deployment.

## Setting the admin password

You have to set the admin password before you can use it to connect. To set the password through the {{site.data.keyword.cloud_notm}} dashboard, select _Manage_ from the left sidebar and open the management panel for your service. Open the _Settings_ tab, and use the _Change Password_ panel to set a new admin password.

You can also set the admin user password using the API. Send a `PATCH` request to the  `https://api.{region}.databases.cloud.ibm.com/v4/ibm/deployments/{id}/users/{username}` endpoint. Use your deployment's id (CRN) and `admin` for the username. Specify the new password in the body of the request. For example,
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
More information is in the [API Reference](https://pages.github.ibm.com/compose/apidocs/apiv4doc-static.html#operation/changeUserPassword)

## Getting Connection Strings

Connection information can be accessed in a few different ways.
- The {{site.data.keyword.cloud_notm}} CLI databases plugin provides the admin connection string in URI format with the command: `ibmcloud dbs deployment-connections "your-service-name"`
- The API will return connection information from a `GET` request to the `https://api.{region}.databases.cloud.ibm.com/v4/ibm/deployments/{id}/users/admin/connections` endpoint.
- Any PostgreSQL user can be added to the _Service Credentials_ panel by providing its authentication information in the _Add Inline Configuration Parameters_ field. For example, to add the admin user to _Service Credentials_ use  `{"existing_credentials":{"username":"admin","password":"your_admin_password_here"}}`. This will make the admin connection string information available in the _Service Credentials_ panel.

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

## Connecting with psql

You can use psql, the command-line tool for PostgreSQL, to administrate your databases. You can connect to `psql` from the {{site.data.keyword.cloud_notm}} CLI databases plugin with the admin user with `ibmcloud dbs deployment-connections "your-service-name" -u admin --start`. It will prompt for the admin password.

Any PosgreSQL user's `psql` connection information is available through the API and can also be made available through the _Service Credentials_ panel. When retrieved through the API or the _Service Credentials_ panel, `psql` connection information is in "cli" field. The below table describes the sub-fields of connection information.

Field Name|Description
----------|-----------
`arguements`|The information that is passed as arguemnets to the `psql` command,
`bin`|The package that this information is intended for; in this case `psql`.
`certificate`|A self-signed certificate that is used to confirm that an application is connecting to the appropriate server. This is base64 encoded. You need to decode the key before using it.
`composed`|A formatted `psql` command to establich a connection to your deployment.
`environment`|`psql` arguements that can be set and pulled from the environment.
`type`|The type of package that uses this connection information; in this case `cli`. 
{: caption="Table 2. `psql` connection information" caption-side="top"}