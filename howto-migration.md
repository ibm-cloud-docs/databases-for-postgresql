---
copyright:
  years: 2017,2018
lastupdated: "2018-09-27"
---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}
{:tip: .tip}

# Migrating data from another PostgreSQL database

Various options exist to migrate data from existing PostgreSQL databases to {{site.data.keyword.databases-for-postgresql_full}}. We focus on the simplest and most effective. To get started, you need PostgreSQL installed locally so you have the `psql` and `pg_dump` tools. And while not strictly required, the {{site.data.keyword.databases-for}} CLI makes it easier to connect and restore to a new {{site.data.keyword.databases-for-postgresql}} deployment. 

## pg_dump

On your source database run `pg_dump` to create an SQL file, which can be used to re-create the database. At a minimum, `pg_dump` takes a host name (`-h` flag), port number (`-p` flag), database name (`-d` flag), user name (`-U` flag), and a file (or directory name) to write the dump to (`-f` flag). 

For example, the following command dumps the PostgreSQL "compose" database that is hosted on sl-eu-lon-2-portal.4.dblayer.com, port 17980, using the admin user and save the results in `dump.sql`.

```shell
pg_dump -h sl-eu-lon-2-portal.4.dblayer.com -p 17980 -d compose -U admin -f dump.sql
```

The `pg_dump` command has many options and it is recommended that you [consult the official documentation](https://www.postgresql.org/docs/9.6/static/backup-dump.html) and [command reference](https://www.postgresql.org/docs/9.6/static/app-pgdump.html) for a fuller view of its capabilities.

## Restoring pg_dump's output

The resulting output of `pg_dump` can then be uploaded into a new {{site.data.keyword.databases-for-postgresql}} deployment. As the output is SQL, it can simply be sent to the database through the `psql` command. We recommend that imports be performed with the admin user. 

See the [Connecting with `psql`](./admin-connecting.html) for details on how to connect as admin using `psql`. To connect with the `psql` command, you need the admin user's connection string and the TLS certificate. The certificate needs to be decoded from the base64 and stored as an arbitrary local file.  To import the previously created `dump.sql` into a database deployment named `example-psql`, the `psql` command can be called with `-f dump.sql` as a parameter. The parameter tells `psql` to read and execute the SQL statements in the file. The command looks something like:

```shell
PGPASSWORD=yourpasswordhere PGSSLROOTCERT=cert.crt psql 'host=c7798cf6-e5d2-4513-b17f-3d3fa67d8291.8f7bfd8f3faa4218aec56e069eb46187.databases.appdomain.cloud port=32484 dbname=ibmclouddb user=admin sslmode=verify-full' -f dump.sql
```

As noted in that [Connecting with `psql`](./admin-connecting.html) documentation, the {{site.data.keyword.databases-for}} CLI plug-in simplifies connecting. The previous `psql` import can be performed as:

```shell
ibmcloud cdb deployment-connections example-psql -s -- -f dump.sql
```

The command automatically uses the admin user, if no user is specified. It also interactively prompts for the password. The TLS certificate is automatically retrieved and used. The `-s` starts `psql` (or whatever command has been configured) once the details are established from the API. Anything after the `--` is passed to the command.

While the restore process is running, it emits a number of messages about changes it is making to the database deployment.