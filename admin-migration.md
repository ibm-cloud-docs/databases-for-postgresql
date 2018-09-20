---
copyright:
  years: 2017,2018
lastupdated: "2018-09-20"
---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}
{:tip: .tip}

# Migrating data from another PostgreSQL database

Various options exist to migrate data from existing PostgreSQL databases to {{site.data.keyword.databases-for-postgresql_full}}. We will focus on the simplest and most effective.

## pg_dump

You will need:

* PostgreSQL installed locally so you have the `psql` and `pg_dump` tools.
* The {{site.data.keyword.databases-for}} CLI installed.

On your source database run `pg_dump` to create an SQL file which can be used to recreate the database. At a minimum, `pg_dump` takes a host name (`-h` flag), port number (`-p` flag), database name (`-d` flag), user name (`-U` flag) and a file (or directory name) to write the dump to (`-f` flag). 

For example, the following command will dump the PostgreSQL "compose" database hosted on sl-eu-lon-2-portal.4.dblayer.com, port 17980, using the admin user and save the results in `dump.sql`.

```shell
pg_dump -h sl-eu-lon-2-portal.4.dblayer.com -p 17980 -d compose -U admin -f dump.sql
```

The `pg_dump` command has many options and it is recommended that you [consult the official documentation](https://www.postgresql.org/docs/9.6/static/backup-dump.html) and [command reference](https://www.postgresql.org/docs/9.6/static/app-pgdump.html) for a fuller view of its capabilities.

The resulting output of `pg_dump` can then be uploaded into a new {{site.data.keyword.databases-for-postgresql}} deployment. As the output is SQL, it can simply be sent to the database through the `psql` command. We recommend imports be performed with the admin user. 

See the [Administration/Connecting](admin-connecting) for details on how to connect as admin using `psql`. As noted in that section, the {{site.data.keyword.databases-for}} CLI plugin simplifies connecting. For example, to import the previously created `dump.sql` into a database deployment names `example-psql` the following command would work well:

```shell
ibmcloud cdb deployment-connections example-psql -s -- -f dump.sql
```

The command automatically uses the admin user, if no user is specified and will interactivly prompt for the password. While running, the restore process will emit a number of messages about changes it is making to the database deployment.