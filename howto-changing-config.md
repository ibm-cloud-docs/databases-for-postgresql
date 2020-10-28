---
copyright:
  years: 2019
lastupdated: "2019-05-23"

keywords: postgresql, databases

subcollection: databases-for-postgresql

---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}
{:tip: .tip}

# Changing the PostgreSQL Configuration
{: #changing-configuration}

{{site.data.keyword.databases-for-postgresql_full}} allows you to change some of the PosgreSQL configuration settings so you can tune your PostgreSQL databases to your use-case. To make permanent changes to the database configuration, use the {{site.data.keyword.databases-for}} [cli-plugin](/docs/databases-cli-plugin?topic=databases-cli-plugin-cdb-reference#deployment-configuration) or [API](https://{DomainName}/apidocs/cloud-databases-api#change-your-database-configuration) to write the changes to the configuration file for your deployment.

The configuration is defined in a schema. To make a change, you send a JSON object with the settings and their new values to the API or the CLI.  For example, to set the `max_connections` setting to 150, you would supply 
```
{"configuration":{"max_connections":150}}
```
{: .codeblock}
to either the CLI or to the API.

## Using the CLI

You can check the current configuration of your deployment with 
```
ibmcloud cdb deployment-configuration-schema <deployment name or CRN>
```
{: pre}

To change your configuration through the {{site.data.keyword.databases-for}} cli-plugin, use `deployment-configuration` command. 
```
ibmcloud cdb deployment-configuration <deployment name or CRN> [@JSON_FILE | JSON_STRING]
```
{: pre}

The command reads the changes that you would like to make from the JSON object or a file. For more information, see the [reference page](/docs/databases-cli-plugin?topic=databases-cli-plugin-cdb-reference#deployment-configuration).

## Using the API

There are two deployment-configuration endpoints, one for viewing the configuration schema and one for changing the configuration. To view the configuration schema, send a `GET` request to `/deployments/{id}/configuration/schema`.

To change the configuration, send the settings that you would like to change as a JSON object in the request body of a `PATCH` request to `/deployments/{id}/configuration`.

For more information, see the [API Reference](https://cloud.ibm.com/apidocs/cloud-databases-api#change-your-database-configuration).


## Available Configuration settings

### Memory Settings

[`shared_buffers`](https://www.postgresql.org/docs/current/runtime-config-resource.html#GUC-SHARED-BUFFERS)
  - Default - `32000` (number of 8 KiB buffers, or about 262 MB) 
  - Recommended max value: 1/4 of available RAM
  - Restarts database? - **Yes** 
  - Notes - The recommended memory allocation for shared_buffers is 1/4 of the deployment's RAM. **Warning: Setting `shared_buffers` any higher can result in memory issues that cause the database to crash, and might decrease the performance of your Database as data is most likely buffered by the OS already.** Setting `shared_buffers` equal, close to equal, or higher than the amount of allocated memory prevents the database from starting. The setting specifies the number of 8 KiB shared memory buffers. For example, 1 GB of `shared_buffers` space is `1048576 KiB`, and (`1048576 KiB / 8 KiB`) is `131072` buffers.  Your deployment can make use of additional RAM for caching and performance, even without allocating it to `shared_buffers`. You do not have to configure the database to use all of the allocated RAM in order for your deployment to use it.

### General Settings

[`max_connections`](https://www.postgresql.org/docs/current/runtime-config-connection.html#GUC-MAX-CONNECTIONS)
  - Default - 115
  - Restarts database? - **YES**
  - Notes - [You might need to scale before you increase max connections.](/docs/databases-for-postgresql?topic=databases-for-postgresql-high-availability#connection-limits-ha)

[`max_prepared_transactions`](https://www.postgresql.org/docs/current/runtime-config-resource.html#GUC-MAX-PREPARED-TRANSACTIONS)
  - Default - 0
  - Restarts database? - **YES**
  - Notes - The default value of `0` disables use of [prepared transactions](https://www.postgresql.org/docs/current/sql-prepare-transaction.html) and is strongly recommended unless you need to use them.

[`synchronous_commit`](https://www.postgresql.org/docs/current/wal-async-commit.html)
  - Default - `local`
  - Restarts database? - No
  - Options - `local` or `off`
  - Notes - Setting `synchronous_commit` to `off` increases transaction commit rate at the expense of a loss of committed transactions in the event of an unclean shutdown.

[`effective_io_concurrency`](https://www.postgresql.org/docs/current/runtime-config-resource.html#GUC-EFFECTIVE-IO-CONCURRENCY)
  - Default - `12`
  - Restarts database - No
  - Notes - It is recommended to leave this setting at the default. Only increase it you have profiled SQL queries and have observed inefficient bitmap heap scans. As [IOPS are tied to disk size](/docs/databases-for-postgresql?topic=databases-for-postgresql-performance#disk-iops), increasing this setting on default or smaller sized disks is also not recommended.

[`deadlock_timeout`](https://www.postgresql.org/docs/current/runtime-config-locks.html)
  - Default - 10000
  - Restarts database - No
  - Options - Minimum value of 100
  - Notes - The number of milliseconds to wait before checking for deadlock and the duration where lock waits are logged. Logs available through the [logging integration](/docs/databases-for-postgresql?topic=cloud-databases-logging). Setting this number too low negatively impacts performance.

### WAL Settings

[`archive_timeout`](https://www.postgresql.org/docs/current/runtime-config-wal.html)
  - Default - `1800`
  - Restarts database - No
  - Options - Minimum value of 300
  - Notes - The number of seconds to wait before forcing a switch to the next WAL file. If the amount of seconds has passed and if there has been database activity, the server switches to a new segment. Effectively limits the amount of time data can remain unarchived.

[`log_min_duration_statement`](https://www.postgresql.org/docs/current/runtime-config-logging.html)
  - Default - `100`
  - Restarts database - No
  - Options - Minimum value of 100
  - Notes - Statements that take longer than the specified number of milliseconds are logged.


The next three settings `wal_level`, `max_replication_slots` and `max_wal_senders` enable use of the [`wal2json` logical decoding plugin](/docs/databases-for-postgresql?topic=databases-for-postgresql-wal2json). Anyone not using this plugin should leave these settings at the default.

[`wal_level`](https://www.postgresql.org/docs/current/runtime-config-wal.html)
  - Default - `hot_standby`
  - Restarts database - **YES**
  - Notes - Controls WAL level. Allowed values are `hot_standby` or `logical`. Set to logical to use logical decoding. If you are not using logical decoding, using `logical` increases the WAL size, which has several disadvantages and no real advantage. If you check your configuration using `SHOW wal_level;` in `psql`, note that as of version 9.6 the value `hot_standby` is mapped to `replica`.

[`max_replication_slots`](https://www.postgresql.org/docs/current/runtime-config-replication.html)
  - Default - `10`
  - Restarts database - **YES**
  - Notes - The maximum number of simultaneously defined replication slots. The default and minimum number of slots is 10. 20 slots are reserved for internal use by your deployment for High-Availability (HA) purposes. To use slots, you need to set the value above 20 and have 1 slot per consumer. It is recommended to add one additional slot over the minimum per expected consumer. Using `wal2json` and not increasing `max_replication_slots` can impact HA and read-only replicas. If you are not using `wal2json`, you should leave this setting at the default.

[`max_wal_senders`](https://www.postgresql.org/docs/current/runtime-config-replication.html)
  - Default - `12`
  - Restarts database - **YES**
  - Notes - The maximum number of simultaneously running WAL sender processes. The default and minimum is 12. One `wal_sender` per consumer is required.  20 slots are reserved for internal use by your deployment for High-Availability (HA) purposes. You need to set the value above 20 and it is recommended to add one additional `wal_sender` over the minimum per expected consumer. Using `wal2json` and not increasing `max_wal_senders` can impact HA and read-only replicas. If you are not using `wal2json`, you should leave this setting at the default.


