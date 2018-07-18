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

# Administrating your PostgreSQL databases

The {{site.data.keyword.databases-for-postgresql_full}} service is provisioned with an admin user intended to allow users to administrate PostgreSQL through it's command-line tool, `psql`.

To get started you will have to:
1. Set the admin credentials
2. Retrieve the admin connection strings and other connection information
3. Decode and save a copy of the self-signed certificate

## Setting the admin password

You have to set the admin password before you can use it to connect. To set the password through the {{site.data.keyword.cloud_notm}} dashboard, select _Manage_ from the left sidebar and open the management panel for your service. Open the _Settings_ tab, and use the _Change Password_ panel to set a new admin password.

You can also set the admin user password using the API. Send a `PATCH` request to the `https://api.{region}.databases.cloud.ibm.com/v4/ibm/deployments/{id}/users/admin` endpoint. For example,
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

## Connecting with `psql`

The {{site.data.keyword.cloud_notm}} CLI cloud databases plugin provides the admin user's connection string in URI format with the command: `ibmcloud dbs deployment-connections "your-service-name"`. Also, you can connect to `psql` from the cloud databases plugin with the admin user with `ibmcloud dbs deployment-connections "your-service-name" -u admin --start`. It will prompt for the admin password.

To retrieve the admin user's connection strings through the API, you can send a GET request to `https://api.{region}.databases.cloud.ibm.com/v4/ibm/deployments/{id}/users/admin`. The JSON response includes the strings to connect to `psql` in "cli" field. The below table describes the sub-fields of connection information.

Field Name|Description
----------|-----------
`arguements`|The information that is passed as arguemnets to the `psql` command,
`bin`|The package that this information is intended for; in this case `psql`.
`certificate`|A self-signed certificate that is used to confirm that an application is connecting to the appropriate server. This is base64 encoded. You need to decode the key before using it.
`composed`|A formatted `psql` command to establich a connection to your deployment.
`environment`|`psql` arguements that can be set and pulled from the environment.
`type`|The type of package that uses this connection information; in this case `cli`. 
{: caption="Table 2. `psql` connection information" caption-side="top"}

## Using the self-signed certificate



