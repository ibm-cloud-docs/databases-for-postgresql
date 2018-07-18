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

## Getting Connection Strings

Connection information can be accessed in a few different ways.
- The {{site.data.keyword.cloud_notm}} CLI databases plugin provides ----------- connection string in URI format with the command: `ibmcloud dbs deployment-connections "your-service-name"`
- The API will return connection information from a `GET` request to the `https://api.{region}.databases.cloud.ibm.com/v4/ibm/deployments/{id}/users/{username}/connections` endpoint.
- Any PostgreSQL user can be added to the _Service Credentials_ panel by providing its authentication information in the _Add Inline Configuration Parameters_ field. For example, using `{"existing_credentials":{"username":"example_user","password":"your_password_here"}}`, will make the connection string information for "example_user" available in the _Service Credentials_ panel.

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
 
