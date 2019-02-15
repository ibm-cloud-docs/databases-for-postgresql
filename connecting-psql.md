---
copyright:
  years: 2017,2018
lastupdated: "2018-12-03"
---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}
{:tip: .tip}


# Connecting with `psql`
{: #connecting-psql}

You can access your PostgreSQL database directly from its command line client, `psql`. You can use `psql` for direct interaction and monitoring of the data structures that are created within the database. It is also useful for testing and monitoring the queries and performance, installing and modifying scripts, and other management activities.

The admin user comes with the PostgreSQL default role [`pg_monitor`](https://www.postgresql.org/docs/10/static/default-roles.html), allowing access to PostgreSQL monitoring views and functions. By default, the admin user does not have permissions on objects that are created by other users.

You have to set the admin password before you use it to connect to the database. For more information, see the [Setting the Admin Password](/docs/services/databases-for-postgresql?topic=databases-for-postgresql-admin-password) page.
{: .tip}

## Installing `psql`

Install the command line client for PostgreSQL, `psql`. To use `psql`, the PostgreSQL client tools need to be installed on the local system. They can be installed with the full PostgreSQL package that is provided from postgresql.org, or from your operating systems packages. For more information about `psql`, see the [PostgreSQL documentation](https://www.postgresql.org/docs/current/static/app-psql.html).

## `psql` Connection Strings

{{site.data.keyword.databases-for-postresql_full}} provides connection strings specifically for CLI clients. The information that you need to make a connection with `psql` is in the "cli" section of your [connection strings](/docs/services/databases-for-postgresql?topic=databases-for-postgresql-connection-strings). The table contains a breakdown for reference.

Field Name|Index|Description
----------|-----|-----------
`Bin`||The recommended binary to create a connection; in this case it is `psql`.
`Composed`||A formatted command to establish a connection to your deployment. The command combines the `Bin` executable, `Environment` variable settings, and uses `Arguments` as command line parameters.
`Environment`||A list of key/values you set as environment variables.
`Arguments`|0...|The information that is passed as arguments to the command shown in the Bin field.
`Certificate`|Base64|A self-signed certificate that is used to confirm that an application is connecting to the appropriate server. It is base64 encoded.
`Certificate`|Name|The allocated name for the self-signed certificate.
`Type`||The type of package that uses this connection information; in this case `cli`. 
{: caption="Table 1. `psql`/`cli` connection information" caption-side="top"}

* `0...` indicates that there might be one or more of these entries in an array.

## Connecting

The `ibmcloud cdb deployment-connections` command handles everything that is involved in creating a command line client connection. For example, to connect to a deployment named  "example-postgres", use the following command.

```
ibmcloud cdb deployment-connections example-postgres --start
```
Or
```
ibmcloud cdb cxn example-postgres -s
```

The command prompts for the admin password and then runs the `psql` command line client to connect to the database.

If you have not installed the cloud databases plug-in, connect to your PostgreSQL databases using `psql` by giving it the "composed" connection string. It provides environment variables `PGPASSWORD` and `PGSSLROOTCERT`. Set `PGPASSWORD` to the admin's password and `PGSSLROOTCERT` to the path or file name for the self-signed certificate. 

```
PGPASSWORD=$PASSWORD PGSSLROOTCERT=0b22f14b-7ba2-11e8-b8e9-568642342d40 psql 'host=4a8148fa-3806-4f9c-b3fc-6467f11b13bd.8f7bfd7f3faa4218aec56e069eb46187.databases.appdomain.cloud port=32325 dbname=ibmclouddb user=admin sslmode=verify-full'
```
## Using the self-signed certificate

1. Copy the certificate information from the Base64 field of the connection information. 
2. Decode the Base64 string into text and save it to a file. (You can use the Name that is provided or your own file name).

### CLI plug-in support for the self-signed certificate

You can display the decoded certificate for your deployment with the CLI plug-in with the command `ibmcloud cdb deployment-cacert "your-service-name"`. It decodes the base64 into text. Copy and save the command's output to a file and provide the file's path to the `ROOTCERT` environment variable.
