---
copyright:
  years: 2017, 2025
lastupdated: "2025-05-13"

keywords: postgresql, databases, postgres connections string, postgresql connection string

subcollection: databases-for-postgresql

---

{{site.data.keyword.attribute-definition-list}}


# Getting connection strings
{: #connection-strings}

## Getting connection strings in the UI
{: #connection-strings-ui}
{: ui}

To connect to {{site.data.keyword.databases-for-postgresql_full}}, you need some users and connection strings. Connection Strings for your deployment are displayed on the _Overview_ page, in the _Endpoints_ panel. 

![Endpoints panel on the Dashboard Overview](images/getting-started-endpoints-panel.png){: caption="Endpoints panel on the Dashboard Overview" caption-side="bottom"}

A {{site.data.keyword.databases-for-postgresql}} deployment is provisioned with an admin user, and after [setting the admin password](/docs/databases-for-postgresql?topic=databases-for-postgresql-user-management&interface=ui#user-management-set-admin-password-ui), you can use its connection strings to connect to your deployment.
{: .tip}

## Getting connection strings in the CLI
{: #connection-strings-cli}
{: cli}

Grab connection strings from the [CLI](/docs/databases-cli-plugin?topic=databases-cli-plugin-cdb-reference#deployment-connections).
```sh
ibmcloud cdb deployment-connections example-deployment -u <NEW_USERNAME> [--endpoint-type <ENDPOINT_TYPE>]
```
{: pre}

Full connection information is returned by the `ibmcloud cdb deployment-connections` command with the `--all` flag. To retrieve all the connection information for a deployment named "example-deployment", use the following command.
```sh
ibmcloud cdb deployment-connections example-deployment -u <NEW_USERNAME> --all [--endpoint-type <ENDPOINT_TYPE>]
```
{: pre}

If you don't specify a user, the `deployment-connections` commands return information for the admin user by default. If you don't specify an endpoint type, the connection string returns the public endpoint by default. If your deployment has only a private endpoint, you must specify `--endpoint-type private` or the commands return an error. The user and endpoint type is not enforced. You can use any user on your deployment with either endpoint (if both exist on your deployment).

To use the `ibmcloud cdb` CLI commands, you must [install the {{site.data.keyword.databases-for}} plug-in](/docs/cloud-databases?topic=cloud-databases-icd-cli).
{: .tip}

## Getting connection strings in the API
{: #connection-strings-api}
{: api}

To retrieve user's connection strings from the API, use the [`/users/{userid}/connections`](/apidocs/cloud-databases-api/cloud-databases-api-v5#getconnection) endpoint. You must specify in the path which user and which type of endpoint (public or private) is to be used in the returned connection strings. The user and endpoint type is not enforced. You can use any user on your deployment with either endpoint (if both exist on your deployment).
```sh
curl -X GET -H "Authorization: Bearer $APIKEY" 'https://api.{region}.databases.cloud.ibm.com/v4/ibm/deployments/{id}/users/{userid}/connections/{endpoint_type}'
```
{: pre}

## Connection string breakdown
{: #connection-string-breakdown}

### The PostgreSQL section
{: #postgres-section}

The "PostgreSQL" tab contains information that is suited to applications that make connections to PostgreSQL.

| Field Name | Index | Description |
| ---------- | ----- | ----------- |
| `Type` | | Type of connection - for PostgreSQL, it is "URI" |
| `Scheme` | | Scheme for a URI - for PostgreSQL, it is "postgresql" |
| `Path` | | Path for a URI - for PostgreSQL, it is the database name. The default is `ibmclouddb`. |
| `Authentication` | `Username` | The username that you use to connect. |
| `Authentication` | `Password` | A password for the user - might be shown as `$PASSWORD` |
| `Authentication` | `Method`|How authentication takes place; "direct" authentication is handled by the driver. |
| `Hosts` | `0...` | A hostname and port to connect to |
| `Composed` | `0...` | A URI combining Scheme, Authentication, Host, and Path |
| `Certificate` | `Name` | The allocated name for the service proprietary certificate for database deployment |
| `Certificate` | Base64 | A base64 encoded version of the certificate. |
{: caption="postgresql/URI connection information" caption-side="top"}

* `0...` indicates one or more of these entries in an array.

### The CLI section
{: #cli-section}
{: cli}

The "CLI" section contains information that is suited for connecting with `psql`.

| Field Name | Index | Description |
| ---------- | ----- | ----------- |
| `Bin` | | The binary to create a connection; in this case it is `psql`. |
| `Composed` | | A formatted command to establish a connection to your deployment. The command combines the `Bin` executable, `Environment` variable settings, and uses `Arguments` as command line parameters. |
| `Environment` | | A list of key/values you set as environment variables. |
| `Arguments` | 0... | The information that is passed as arguments to the command shown in the Bin field. |
| `Certificate` | Base64 | A service proprietary certificate that is used to confirm that an application is connecting to the appropriate server. It is base64 encoded. |
| `Certificate` | Name | The allocated name for the service proprietary certificate. |
| `Type` | | The type of package that uses this connection information; in this case `cli`.  |
{: caption="psql/cli connection information" caption-side="top"}

* `0...` indicates that there might be one or more of these entries in an array.
