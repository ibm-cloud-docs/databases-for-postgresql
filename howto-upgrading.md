---
copyright:
  years: 2020, 2026
lastupdated: "2026-10-07"

keywords: postgresql, databases, upgrading, major versions, postgresql new deployment, postgresql database version, postgresql major version

subcollection: databases-for-postgresql

---

{{site.data.keyword.attribute-definition-list}}

# Upgrading to a new major version
{: #upgrading}

Choose an upgrade path:

- **[In-place major version upgrade (IPMVU)](#upgrading-in-place):** Upgrade the existing deployment and retain its connection strings.
- **[Upgrade a read-only replica](#upgrading-replica):** Replicate data to a separate deployment, then promote and upgrade it.
- **[Restore from backup](#backup-restore):** Create a deployment on a supported target version from an existing backup.

For the last two paths, the new deployment does not receive changes that reach the source after the promotion or after the backup. Rehearse and validate the new version first. Before the final switch, stop writes to the source. Then, promote the replica or take the backup, and switch your application's connection details to the new deployment.

Find versions available for new deployments in the [catalog](https://cloud.ibm.com/databases/databases-for-postgresql/create), with [`ibmcloud cdb deployables-show`](/docs/cli?topic=cli-cdb-reference#deployables-show), or through the [`/deployables` API](/apidocs/cloud-databases-api/cloud-databases-api-v5#listdeployables).

In API examples, replace `{region}` with the deployment's region, `{id}` with the deployment's URL-encoded Cloud Resource Name (CRN), and `<IAM_TOKEN>` with your Identity and Access Management (IAM) access token.

## Requirements for upgrading
{: #upgrading-reqs}

### Application compatibility
{: #app-dependencies}

Test your chosen upgrade path on a representative staging deployment before production. For an IPMVU rehearsal, restore a recent backup to the source PostgreSQL version. A deployment has two or three data members: the primary and one or two high-availability (HA) members. Scale the restored deployment to the same number of members as production, because the [conversion phase](#upgrading-in-place-availability) takes longer with more HA members. Make sure that the restored deployment has a completed backup, and then upgrade it to your intended target. Check application queries, background jobs, drivers, extensions, and connection pools. Record performance, and how long your applications see read-only errors and connection failures, for comparison afterward.

Review the [release notes](#changelog-postgres) for every major version after your source version, up to and including your target version. Also review the [role privilege changes](#admin_user_issues).

The upgrade does not keep commit timestamps. After the upgrade, `pg_xact_commit_timestamp()` returns `NULL` for every transaction that committed before the upgrade. If your applications read commit timestamps, copy the values that you need into a table column before you upgrade.

### Extensions and other objects
{: #extensions-objects}

Complete the applicable preparation in the following sections before upgrading. For an IPMVU, prepare the deployment that you upgrade. For the read-only replica and restore paths, prepare the source deployment before you create the replica or take the backup. You cannot change a read-only replica, and a restore upgrades the data in the same operation. Plan for any application dependencies on objects that you remove.

#### Extensions
{: #extensions}

##### `pg_repack`
{: #pg_repack}

`pg_repack` does not block an upgrade. You do not need to drop or re-create it before or after the upgrade.

##### `old_snapshot`
{: #old_snapshot}

The [`old_snapshot`](https://www.postgresql.org/docs/16/oldsnapshot.html){: external} module is not available in PostgreSQL 17 and later. When you upgrade from PostgreSQL 16 or earlier to PostgreSQL 17 or later, IPMVU removes `old_snapshot` from each database before read-only mode begins. If any object depends on `old_snapshot`, the upgrade fails. The removal is not reversed if the upgrade fails later.

If `old_snapshot` is installed in any database, contact support to identify dependent objects before you upgrade. For the other upgrade paths, contact support to arrange its removal.

##### `anon`
{: #anon}

If `anon` version 2 is installed, complete these steps in each database where it is installed. Run them as the owner of the masked tables, such as `admin`. For an IPMVU, `anon` version 2 makes the upgrade fail during the conversion phase, while the database is unavailable, and every retry fails the same way. `anon` version 3 does not need these steps. Self-check query 10 shows the installed version in `extversion`.

Before you remove the masking rules, block database access for the masked roles, which must see only masked data. Keep their access blocked until you restore and verify the masking rules and role labels after the upgrade.
{: important}

1. Record the masking rules and masked roles so that you can restore them after the upgrade.

    ```sql
    SELECT objtype, objname, label
    FROM pg_catalog.pg_seclabels
    WHERE provider = 'anon';
    ```
    {: pre}

2. Remove all masking rules.

    ```sql
    SELECT anon.remove_masks_for_all_columns();
    ```
    {: pre}

3. Remove the masked label from each role listed in step 1. On PostgreSQL 16 and later, this action requires the `ADMIN` option on the role. For more information, see [Role privilege issues during version upgrades](#admin_user_issues).

    ```sql
    SECURITY LABEL FOR anon ON ROLE <role_name> IS NULL;
    ```
    {: pre}

4. Drop the `anon` extension with the cascade option. The cascade option also drops objects that depend on `anon`, such as masking views and functions. Record their definitions so that you can recreate them after the upgrade.

    ```sql
    DROP EXTENSION anon CASCADE;
    ```
    {: pre}

5. After the upgrade, re-enable `anon`, reapply the recorded masking rules and role labels, and verify masking before you restore access for the masked roles.

##### `PostGIS`
{: #PostGIS}

If you use PostGIS, upgrade it in each database that uses it before you upgrade PostgreSQL:

```sql
SELECT postgis_extensions_upgrade();
```
{: pre}

Verify the extension version:

```sql
SELECT postgis_full_version();
```
{: pre}

##### `earthdistance`
{: #earthdistance}

An IPMVU fails during the conversion phase if an index or a stored generated column uses a function of `earthdistance` version 1.1, such as `ll_to_earth()`. This also applies when the object uses it through your own SQL function. PostgreSQL 14 and 15 provide only version 1.1. Check with [self-check query 13](#upgrading-in-place-self-checks) in each database, and then handle each listed object:

- On PostgreSQL 16 or 17, update the extension before you upgrade:

    ```sql
    ALTER EXTENSION earthdistance UPDATE;
    ```
    {: pre}

- On PostgreSQL 14 or 15, if `used_function` is one of your own functions, add a `search_path` setting to it:

    ```sql
    ALTER FUNCTION <used_function> SET search_path = ibm_extension, pg_catalog;
    ```
    {: pre}

- On PostgreSQL 14 or 15, if `used_function` is an `earthdistance` function, drop the listed index before you upgrade. To keep the values of a listed generated column, convert it to a regular column with `ALTER TABLE <table_name> ALTER COLUMN <column_name> DROP EXPRESSION;`. After the upgrade, run `ALTER EXTENSION earthdistance UPDATE;`, and then re-create what you removed. If your target is PostgreSQL 15, the update has no effect, and the objects that you re-create block your next upgrade in the same way. To avoid this, upgrade to PostgreSQL 16 or later.

##### `pgRouting`
{: #pgrouting}

If your deployment runs PostgreSQL 16 or 17 and you upgrade to PostgreSQL 18, check each database where `pgRouting` is installed. PostgreSQL 18 deployments do not include the `pgRouting` 3.5 library. An IPMVU to PostgreSQL 18 fails its compatibility checks, before read-only mode begins, while any function uses that library. Expect no rows:

```sql
SELECT DISTINCT p.probin
FROM pg_catalog.pg_proc AS p
WHERE p.probin = '$libdir/libpgrouting-3.5';
```
{: pre}

If the query returns a row, update `pgRouting` to version 4.0.1 before you upgrade, and then run the query again:

```sql
ALTER EXTENSION pgrouting UPDATE TO '4.0.1';
```
{: pre}

`pgRouting` 4.0 removes several functions, such as `pgr_createTopology`, `pgr_analyzeGraph`, and `pgr_nodeNetwork`, and changes others. Test your applications with version 4.0.1 on a staging deployment first. If they need the removed functions, contact support before you upgrade to PostgreSQL 18. From PostgreSQL 16, you can also upgrade to PostgreSQL 17, which includes the 3.5 library.

#### SQL functions in indexes, partition keys, and generated columns
{: #sql-function-wrappers}

During the conversion phase, an IPMVU re-creates the definition of every index, partition key, and stored generated column with an empty search path. This step fails if one of these objects uses a `LANGUAGE sql` function with a string body that names an object outside `pg_catalog` without its schema, directly or through a domain check constraint. Extended statistics have the same effect on a table that has a foreign key. In an index, a call that names the `ibm_extension` schema, where extensions are installed, also fails. The failure happens while the database is unavailable, and every retry fails the same way.

Run [self-check query 12](#upgrading-in-place-self-checks) in each database. For each listed function, add a `search_path` setting that names every schema that its body uses. Alternatively, replace its string body with an SQL-standard body. Run these statements as the function's owner. For example:

```sql
ALTER FUNCTION f_unaccent(text) SET search_path = ibm_extension, pg_catalog;
```
{: pre}

```sql
CREATE OR REPLACE FUNCTION f_unaccent(text) RETURNS text
  LANGUAGE sql IMMUTABLE PARALLEL SAFE STRICT
  RETURN ibm_extension.unaccent('ibm_extension.unaccent', $1);
```
{: pre}

Keep each function's results and other attributes unchanged, because indexes and generated columns store its results, and partitioned tables route rows by them. With a `search_path` setting, PostgreSQL no longer expands the function into the queries that call it, so those queries can be slower. An SQL-standard body does not protect the string-body functions that it calls, so fix those functions too. Run the query again until it returns no rows. If it lists a function that you do not own, contact support before you upgrade.

If you upgrade from PostgreSQL 16 or earlier to PostgreSQL 17 or later, also run [self-check query 17](#upgrading-in-place-self-checks). PostgreSQL 17 and later run maintenance commands with a restricted search path. A function in any language that an index, a materialized view, or extended statistics use then needs its own `search_path` setting. Otherwise, after the upgrade, `ANALYZE`, `REINDEX`, `CLUSTER`, `VACUUM FULL`, and `REFRESH MATERIALIZED VIEW` fail for the affected objects. Set the search path of each listed function that names objects without their schema, as in the previous example. For more information, see the [PostgreSQL 17 migration notes](https://www.postgresql.org/docs/17/release-17.html#RELEASE-17-MIGRATION){: external}.

#### Objects that use removed system names
{: #removed-system-objects}

Each major version removes or renames some system catalog columns, functions, and configuration parameters. For example, PostgreSQL 17 removed the checkpoint columns from `pg_stat_bgwriter`, and PostgreSQL 16 renamed the `force_parallel_mode` parameter to `debug_parallel_query`. The prechecks do not detect objects that use these names:

- If a view, a materialized view, or an aggregate uses a removed name, an IPMVU fails during the conversion phase. A function with an SQL-standard body and a function, role, or database setting of a removed parameter have the same effect.
- Other functions and procedures, `pg_cron` jobs, and application queries that use a removed name fail when they first run on the new version. This can occur before writes resume.

Run [self-check query 14](#upgrading-in-place-self-checks) in each database. Change or drop each listed object whose `removed_in` version is at or below your target version. Check role and database settings with self-check query 7. Test your application and monitoring queries on a staging deployment of the target version.

#### Logical replication slots
{: #replication-slots}

IPMVU fails its prechecks while any logical replication slot exists, including inactive slots and slots that `wal2json` uses. You cannot recover a deleted slot, and a new slot does not include the changes that the old slot retained. Before you upgrade:

1. Record each slot's name, plug-in, and database with [self-check query 2](#upgrading-in-place-self-checks).
2. Pause source writes until you restore consumers after the upgrade, or plan to resynchronize downstream data.
3. Let consumers process pending changes, and then stop them. Keep them stopped, with automatic reconnection turned off, until the task shows **Completed**.
4. Delete each slot with the [`ibmcloud cdb postgresql replication-slot-delete`](/docs/cli?topic=cli-cdb-reference#postgresql-replication-slot-delete) command or the [Delete a logical replication slot](/apidocs/cloud-databases-api/cloud-databases-api-v5#deletelogicalreplicationslot) API. The `admin` user cannot drop replication slots with SQL.

    ```sh
    ibmcloud cdb postgresql replication-slot-delete <NAME|CRN> <SLOT_NAME>
    ```
    {: pre}

5. After the upgrade, recreate the slots as described in [Configuring `wal2json`](/docs/databases-for-postgresql?topic=databases-for-postgresql-wal2json).

`wal2json` is a logical decoding output plug-in, not an extension that you create with `CREATE EXTENSION`.

#### Logical subscriptions
{: #replication-subscriptions}

A logical subscription receives changes from a publisher. Its replication slot is on the publisher, so the slot query does not list it.

If your deployment runs PostgreSQL 16 or earlier, IPMVU fails its prechecks while any logical subscription exists, including disabled subscriptions. Upgrades from these versions do not preserve a subscription's synchronization state. Before you upgrade, agree with the publisher's owner how to remove and recreate each subscription, and how to resynchronize the data. Then, delete the subscriptions with the [subscriber functions](/docs/databases-for-postgresql?topic=databases-for-postgresql-logical-replication#subscriber-functions). The `delete_subscription` function also drops the subscription's slot on the publisher, and it fails if the publisher is unreachable or the slot no longer exists. In that case, run `disable_subscription` and then `subscription_slot_none` before `delete_subscription`. Then, ask the publisher's owner to drop the slot, because it keeps retaining transaction logs on the publisher.

If your deployment runs PostgreSQL 17 or later, subscriptions can remain, but they must meet the target version's [logical replication upgrade requirements](https://www.postgresql.org/docs/18/logical-replication-upgrade.html){: external}. The upgrade supports at most 10 subscriptions. With more, it fails its compatibility checks before read-only mode begins. Disable the subscriptions with `disable_subscription` before you upgrade, and enable them with `enable_subscription` after the upgrade completes.

#### `UNLOGGED` tables and sequences
{: #unlogged-objects}

PostgreSQL does not write changes to `UNLOGGED` tables and sequences to the transaction log. It does not replicate them to other members, and it resets them after a failover or a restore. None of the upgrade paths keeps their contents:

- An IPMVU resets them when it restarts PostgreSQL on the new version, and again when it switches the primary role in the completion phase. A failed upgrade can also reset them.
- A promoted read-only replica and a restored backup start with empty `UNLOGGED` tables and reset `UNLOGGED` sequences.

After the upgrade, `UNLOGGED` tables are empty and `UNLOGGED` sequences are reset. Until the task shows **Completed**, any changes that you make to these objects can also be lost. Before you upgrade, use [self-check query 11](#upgrading-in-place-self-checks) to identify these objects in each database and plan how to handle each one.

- To keep the contents of a table, convert it with `ALTER TABLE <table_name> SET LOGGED;`. `SET LOGGED` also converts the table's indexes and the sequences that it owns. Convert a referenced table before converting any tables that reference it through foreign keys. Alternatively, export the contents and reload them after the task shows **Completed**.
- If a sequence generates values for a logged table, convert it with `ALTER SEQUENCE <sequence_name> SET LOGGED;`. Otherwise, stop the applications that use it until the task shows **Completed**. Then, set it past the highest value in use with the `setval` function.

`SET LOGGED` rewrites the table and blocks reads and writes until the operation finishes. It also generates approximately as much transaction log data as the table and its indexes occupy. Convert large tables well before you submit the upgrade, and use self-check query 5 to verify that the transaction logs were archived. After the upgrade, you can convert a table back with `ALTER TABLE <table_name> SET UNLOGGED;`.

For an IPMVU, an `UNLOGGED` table, even an empty one, also makes the service rebuild every HA member, which extends the time that writes are blocked. For more information, see [Availability during an upgrade](#upgrading-in-place-availability). To avoid this rebuild, convert every `UNLOGGED` table, or drop the ones that you no longer need. `TRUNCATE` does not avoid the rebuild. `UNLOGGED` sequences do not cause a rebuild. On PostgreSQL 16 and 17, `SET LOGGED` is itself a bulk operation that can still cause a rebuild, so plan for one.

## In-place major version upgrade
{: #upgrading-in-place}

Complete [preparation](#upgrading-considerations), then use the UI, API, CLI, or Terraform procedure below.

In an IPMVU, the service upgrades the primary in place and upgrades each HA member from its existing data files. When all HA members can reuse their data files, writes are blocked only briefly. The duration depends primarily on the number of database objects and the level of database activity, rather than on the amount of data stored.

Some conditions make the service copy the whole database to the HA members instead, which keeps writes blocked much longer. Other conditions make the upgrade fail after writes are blocked. The prechecks do not detect these conditions, so resolve them before you submit the upgrade. For the list, see [Conditions that the prechecks do not detect](#upgrading-in-place-undetected).

You cannot cancel an IPMVU after you submit it, and you cannot downgrade the deployment in place. The service does not take a data backup before the upgrade, so create and verify an [on-demand backup](/docs/cloud-databases?topic=cloud-databases-dashboard-backups) first.
{: important}

### Availability during an upgrade
{: #upgrading-in-place-availability}

An IPMVU runs in the following phases. Writes stop when the read-only phase begins and return when the service clears read-only mode. Reads are also unavailable during the conversion phase.

| Phase | What happens | Reads | Writes |
| --- | --- | --- | --- |
| Checks | The service waits for pending deployment changes to finish and runs the [prechecks](#upgrading-in-place-prechecks). It checks upgrade compatibility on the primary and the HA members, and suspends automatic failover. | Available | Available |
| Read-only | The service makes new transactions read-only in every database and ends all client sessions. It then waits for the HA members and transaction-log archiving to catch up. | Available | Blocked |
| Conversion | The service stops PostgreSQL on all members and upgrades the primary. Where possible, it upgrades each HA member from that member's existing data files. | Unavailable | Unavailable |
| Restart | The upgraded primary starts in read-only mode, and the HA members start. The service rebuilds any member that cannot reuse its data files from the upgraded primary. | Available | Blocked |
| Writes resume | The service clears read-only mode and ends all client sessions again. | Available | Available |
| Completion | The service waits for every HA member to catch up. Then, the service restarts the members one at a time to complete moving the deployment to the new version. Before restarting the primary, the service switches the primary role to another member, which ends client sessions. The service then removes the upgrade files, queues the post-upgrade backup, and marks the tasks as **Completed**. | Available, except during the switchover | Available, except during the switchover |
{: caption="IPMVU phases and availability" caption-side="bottom"}

Writes resume after at least one HA member has caught up, and every other HA member has caught up or is rebuilding. The service rebuilds an HA member that cannot reuse its data files by copying the whole database to it. If every HA member must be rebuilt, writes stay blocked until the first rebuild finishes. For a large database, this can take hours. A deployment with two data members, which is the default configuration, has only one HA member. As a result, any rebuild blocks writes. HA members cannot reuse their data files in the following situations:

- Any database contains an `UNLOGGED` table, even an empty one. The service then rebuilds every HA member. Check with [self-check query 11](#upgrading-in-place-self-checks), and see [`UNLOGGED` tables and sequences](#unlogged-objects).
- The deployment runs PostgreSQL 16 or 17, and tables grew through bulk operations or many concurrent inserts or updates. Bulk operations include `COPY`, `pg_restore`, `CREATE TABLE AS`, `CREATE MATERIALIZED VIEW`, `REFRESH MATERIALIZED VIEW`, and `ALTER TABLE` commands that rewrite a table, such as `SET LOGGED`. You cannot check for this in advance, so plan for a rebuild.

The service can also rebuild HA members for other reasons that you cannot check in advance. A rehearsal on a staging deployment that you restore from a backup might not show such a rebuild.

A rebuilding HA member keeps its previous data files until the task completes. As a result, it requires free disk space at least equal to the amount of space that its data uses. Before you submit the upgrade, ensure that each data member uses significantly less than half of its available disk space. Check this value by using the [used disk space](/docs/databases-for-postgresql?topic=databases-for-postgresql-monitoring#ibm_databases_for_postgresql_disk_used_bytes) metric. Although the disk space precheck allows up to 90% disk utilization, a rebuilding HA member requires substantially more free space. Otherwise, the rebuild can run out of disk space and fail repeatedly. Writes can then stay blocked for hours before the task fails. Automatic and manual disk scaling wait until the upgrade finishes, so scale the disk before you submit the upgrade. A long rebuild can also make the task fail after writes resume. In both cases, the deployment runs the target version until support completes the upgrade. Allow for a rebuild in your maintenance window.

Writes resume before the task completes. To check whether writes have resumed, run `SHOW default_transaction_read_only;` in a new session. It returns `off` after writes resume. Client sessions end when read-only mode begins, when PostgreSQL stops for the conversion, when writes resume, during the switchover in the completion phase, and when the service restores write access after a failure.

Until the task shows **Completed**, avoid sustained bulk writes, such as large data loads or batch jobs. The completion phase waits until every HA member has replayed all changes. Under a continuous heavy write load, this wait can time out, and the task then fails after writes resume. The risk is higher while an HA member is still being rebuilt. The deployment then runs the target version without a post-upgrade backup, and support must complete the upgrade.

To check whether your write load lets the HA members catch up, run the following query at your planned upgrade time. Repeat it every second for a few minutes, for example with the `psql` command `\watch 1`. The service needs every HA member to show `0` in `replay_lag_bytes` at the same time. If this never occurs, reduce the write load before you submit the upgrade.

```sql
SELECT application_name, state,
       pg_catalog.pg_wal_lsn_diff(pg_catalog.pg_current_wal_lsn(), replay_lsn) AS replay_lag_bytes
FROM pg_catalog.pg_stat_replication
ORDER BY application_name;
```
{: pre}

The upgrade does not refresh optimizer statistics, so queries can choose slow plans until you run `ANALYZE`. Upgrades to PostgreSQL 15, 16, or 17 do not transfer statistics. Upgrades to PostgreSQL 18 transfer most statistics, but not extended statistics that you created with `CREATE STATISTICS`, or statistics that extensions such as PostGIS collect. The `autovacuum` process analyzes a table only after enough writes, and it never analyzes partitioned tables. Do not wait for the task to complete. As soon as `SHOW default_transaction_read_only;` returns `off`, refresh statistics as described in [After the upgrade](#upgrading-in-place-after).

Read-only mode sets a default for new transactions. It does not lock the database. A session that sets `default_transaction_read_only` to `off`, or explicitly starts a read-write transaction, can still write. So can a role whose own `default_transaction_read_only` setting is `off`. Maintenance commands such as `VACUUM`, `ANALYZE`, `REINDEX`, and `CLUSTER` also still run. Read-only mode does not stop a `pg_cron` job that is already running, or an enabled subscription on PostgreSQL 17. Such writes can make the upgrade fail. Before you submit the upgrade, stop processes that override read-only mode. Pause scheduled jobs that write or run maintenance commands, such as `pg_cron` jobs, and wait for running jobs to finish. Resume them as described in [After the upgrade](#upgrading-in-place-after).

Reads can also write transaction logs during read-only mode, for example the first time that queries read rows after a bulk load or a large update. This activity delays the catch-up that the read-only and restart phases wait for, so writes stay blocked longer, and the upgrade can fail. Before you submit the upgrade, run `VACUUM` on tables that you recently loaded or modified significantly.

When the service ends a session, a client that connects directly to the deployment receives SQLSTATE `57P01` (`admin_shutdown`). A write in a read-only transaction fails with SQLSTATE `25006` (`read_only_sql_transaction`). During the conversion phase, a new connection can fail with an authentication error, such as SQLSTATE `28P01`, although your credentials are unchanged. Try the connection again after the conversion. If these errors continue after the task shows **Failed**, contact support. Through a connection pooler, applications can see other connection errors.

Clients that accept only read-write sessions cannot connect while read-only mode is set, so reads are also unavailable to them until writes resume. This applies to `libpq` clients that set `target_session_attrs=read-write`, and to JDBC clients that set `targetServerType=primary` or `master`. To keep reads available, use `target_session_attrs=any`, or `primary` with `libpq` 14 or later. For JDBC, use `targetServerType=any`.

Applications must handle connection failures and read-only errors, reconnect, and retry interrupted transactions only when safe. Connection pooling does not remove these requirements. Application access is not proof that the upgrade task completed.

### Backups and recovery
{: #upgrading-in-place-backups}

The service does not take a data backup before the upgrade. Create and verify a recent [on-demand backup](/docs/cloud-databases?topic=cloud-databases-dashboard-backups) before IPMVU. To return to the source version after an upgrade, restore a pre-upgrade backup or recovery point into a new deployment that runs the source version. Set the version explicitly, because a point-in-time restore without a version creates a deployment that runs the current version. This option requires that the source version is still available for new deployments. Account for changes made since that point.

After a successful upgrade, the service queues a backup of the upgraded deployment. The backup runs separately after the upgrade task completes and might not start immediately. This first backup on the new version copies the whole database, so it can take several hours for a large database. Check its status in **Backups and restore**. If it fails, take an on-demand backup, and contact support if failures continue. A backup failure does not roll back the upgrade.

Point-in-time recovery (PITR) cannot replay transactions across the major version upgrade. You can restore to a point in time before the upgrade task starts, or after the first backup of the upgraded deployment completes. Restores to recovery points between the upgrade and the completion of the post-upgrade backup are not supported. The service rejects some of these recovery points, while restores to others might be accepted but fail during the restore process. This unsupported period includes the time after writes resume. Until the post-upgrade backup completes, restores to the latest available recovery point can also fail even if the service accepts the restore request.

To keep PITR coverage for changes after the upgrade, keep application writes paused until the post-upgrade backup completes. Pause them in your applications. When writes resume, the upgrade resets `default_transaction_read_only` on every database, so a database-level read-only setting does not keep writes paused.

A failed upgrade does not queue a backup. PITR can be unavailable for points between the start of the attempt and the next completed backup. After a failed upgrade, if `SHOW server_version;` reports the source version and the deployment accepts writes, take an on-demand backup. Otherwise, contact support before you take a backup or make other changes.

Pre-upgrade backups and recovery points belong to the earlier version and remain subject to retention limits. For more information, see [PITR](/docs/databases-for-postgresql?topic=databases-for-postgresql-pitr).

The skip-backup options are not supported for PostgreSQL: `skip_backup` in the API, `--skip-backup` in the CLI, and `version_upgrade_skip_backup` in Terraform. Requests that enable them fail. The API and the CLI return an error, and  Terraform plans fail.

### Before you begin
{: #upgrading-considerations}

Apply these requirements to both staging and production:

- Complete the [application, extension, and replication preparation](#upgrading-reqs) and [backup preparation](#upgrading-in-place-backups).
- Resolve the [conditions that the prechecks do not detect](#upgrading-in-place-undetected), and run the [self-check queries](#upgrading-in-place-self-checks).
- Ensure that each data member uses significantly less than half of its available disk space, so that a rebuild of the HA members does not run out of space. For more information, see [Availability during an upgrade](#upgrading-in-place-availability).
- Ensure that the deployment has at least one completed backup. Until then, the console disables the **Upgrade major version** button, and the API rejects the request.
- IPMVU is not available on a read-only replica deployment. To upgrade a replica, [promote it with a version upgrade](#upgrading-replica).
- Promote or delete every read-only replica of the deployment, including replicas in other regions, and wait for each operation to finish. A deleted replica stays associated with the deployment until it is permanently deleted. The upgrade fails its prechecks while any replica is associated with the deployment. Plan for applications that read from these replicas. For more information, see [Read-only replicas and IPMVU](/docs/databases-for-postgresql?topic=databases-for-postgresql-read-only-replicas#read-only-replicas-ipu).
- Run one upgrade at a time. The service rejects a new upgrade request while another upgrade task is queued or running.
- If you set the deployment's `synchronous_commit` configuration to `on`, change it to `local` before you submit the upgrade. Change it back after the task shows **Completed**. With `on`, a failed upgrade can leave every database read-only, or make commits wait, until support recovers the deployment. The deployment's setting is `on` if the following query returns `on` with the source `configuration file`:

    ```sql
    SELECT setting, source
    FROM pg_catalog.pg_settings
    WHERE name = 'synchronous_commit';
    ```
    {: pre}

- Choose a target from your deployment's capabilities. You can upgrade directly to any later version that appears as an upgrade transition, without intermediate versions. A version that you can provision is not necessarily an in-place upgrade target:

    ```sh
    ibmcloud cdb deployment-capability-show <NAME|CRN> versions
    ```
    {: pre}

    In the output, use the `to_version` value of an **Upgrade Transition** entry.

#### Prechecks and preparation
{: #upgrading-in-place-prechecks}

The service runs the following checks before read-only mode begins. If a check fails, the task shows **Failed**, and the deployment stays on its current version without blocking writes. If any database has `default_transaction_read_only` set to `on`, a failed check also resets this setting on every database and ends all client sessions. The task does not report which check failed, so verify each item before you submit the upgrade.

| Check | Passes when | Your action |
| --- | --- | --- |
| Deployment health | Every data member is running, and each HA member is streaming from the primary. | Let scaling, maintenance, and HA member rebuilds finish. Check with self-check query 1. Other tasks do not fail this check, but they delay the start of the upgrade. Check **Recent tasks**, or list tasks with [`ibmcloud cdb deployment-tasks-list`](/docs/cli?topic=cli-cdb-reference#deployment-tasks-list). If this check keeps failing, contact support. |
| Disk space | Each data member uses at most 90% of its allocated disk space. | Scale disk or remove data that you no longer need. Unused logical replication slots can also retain transaction logs. This check allows up to 90%, but a rebuild of the HA members needs each data member to use well under half of its disk. See [Availability during an upgrade](#upgrading-in-place-availability). |
| CPU, memory, and disk I/O | On each data member, the host's 1-minute load average is at most 90% of its CPU count, and host memory use is at most 90%. Disk I/O utilization is also at most 90%. | Upgrade during low activity, or submit the upgrade again later. The service measures load and memory for the whole host, including other workloads on it, and samples disk I/O over a few seconds. The load average also counts processes that wait for disk I/O. Its values can therefore differ from your monitoring metrics, and scaling your deployment might not lower them. If this check keeps failing, contact support. |
| Transaction-log archiving | Archiving is not failing, and no more than 4 GB of transaction logs are waiting for archiving. | Reduce the write rate until the backlog clears. Check with [self-check query 5](#upgrading-in-place-self-checks). The check also fails if an HA member has transaction logs waiting, for example soon after a failover. If this check keeps failing, contact support. |
| Logical replication | No logical replication slot exists. On PostgreSQL 16 and earlier, no logical subscription exists, including disabled subscriptions. | Remove [slots](#replication-slots). On PostgreSQL 16 and earlier, remove [subscriptions](#replication-subscriptions). On PostgreSQL 17, disable them. Check with self-check queries 2 and 3. |
| Read-only replicas and replication clients | The deployment has no associated read-only replica, and no replication client other than the HA members connects to it. Replicas count in any region and state, including replicas that are provisioning or disconnected. | List the associated replicas in the console or with [`ibmcloud cdb deployment-read-replicas`](/docs/cli?topic=cli-cdb-reference#deployment-read-replicas). Promote or delete each one, and wait for each operation to finish. A deleted replica still counts until it is permanently deleted, although the list no longer shows it. See [Read-only replicas and IPMVU](/docs/databases-for-postgresql?topic=databases-for-postgresql-read-only-replicas#read-only-replicas-ipu). Disconnect other replication clients. Self-check query 1 shows only connected clients. |
| Prepared transactions | No prepared transaction exists. | Have the owning application or transaction manager resolve prepared transactions, and prevent new ones until the upgrade completes. Do not commit or roll them back only to clear the check. Check with self-check query 4. |
| Schema export | The schema of each database exports without an error. An export that times out or runs out of lock space does not fail this check. | Rehearse the upgrade on a staging deployment that you restore from a recent backup. This check does not test whether the target version can re-create the schema, so also resolve the [conditions that the prechecks do not detect](#upgrading-in-place-undetected). If this check keeps failing, contact support. |
| Upgrade compatibility | PostgreSQL upgrade compatibility checks pass on the primary and every HA member. | Complete the [extension preparation](#extensions), including the [`pgRouting` check](#pgrouting). On PostgreSQL 17, keep at most 10 [subscriptions](#replication-subscriptions). Check with self-check queries 6, 8, 9, and 10. |
{: caption="IPMVU prechecks" caption-side="bottom"}

#### Conditions that the prechecks do not detect
{: #upgrading-in-place-undetected}

The prechecks do not detect the following conditions. Some of them make the upgrade fail after writes are blocked. If the cause is in the schema, such as a function or a database name, every retry fails the same way until you resolve it. Other conditions make the service copy the whole database to the HA members, which keeps writes blocked much longer, or make the upgrade fail after writes resume. A failure in the conversion or restart phase can also leave the HA members stopped until support restarts them. Resolve or plan for each condition before you submit the upgrade.

| Condition | Effect | What to do |
| --- | --- | --- |
| A database contains an `UNLOGGED` table. | Every HA member is rebuilt, and `UNLOGGED` contents are reset. | Check with self-check query 11. See [`UNLOGGED` tables and sequences](#unlogged-objects). |
| The deployment runs PostgreSQL 16 or 17, and tables grew through bulk operations or many concurrent inserts or updates. | HA members are rebuilt. | Plan the maintenance window and disk space for a rebuild. See [Availability during an upgrade](#upgrading-in-place-availability). |
| A data member uses about half of its disk or more when HA members are rebuilt. | The rebuild runs out of space. Writes can stay blocked for hours before the task fails. | Before you submit the upgrade, scale disk until each data member uses significantly less than 50% of its available disk space. |
| An index, a partition key, a stored generated column, or extended statistics use a `LANGUAGE sql` function with a string body that names objects outside `pg_catalog` without their schema. In an index, naming the `ibm_extension` schema has the same effect. | The conversion fails. | Check with self-check query 12. See [SQL functions in indexes, partition keys, and generated columns](#sql-function-wrappers). |
| An index or a stored generated column uses `earthdistance` version 1.1. | The conversion fails. | Check with self-check query 13. See [`earthdistance`](#earthdistance). |
| A view, an aggregate, or a function with an SQL-standard body uses a system name that the target version removed. Or a function, role, or database setting uses a removed parameter. | The conversion fails. | Check with self-check queries 7 and 14. See [Objects that use removed system names](#removed-system-objects). |
| `anon` version 2 is installed. | The conversion fails. | Complete the [`anon` preparation](#anon). |
| A database name contains a single quotation mark (`'`), a backslash (`\`), an equals sign (`=`), a newline, or a carriage return. | The upgrade fails, in some cases after writes are blocked. | Check with self-check query 15. Rename the database, and update your connection strings. |
| Databases contain many thousands of tables and sequences or, on PostgreSQL 14 and 15, views. | The conversion runs out of lock space and fails. | Check with self-check query 16, and increase `max_locks_per_transaction` if the query shows that you need to. |
| The deployment has very many databases, tables, or large objects. | The conversion exceeds its time limit. The task fails, or stays in progress while the database is unavailable. | Count the tables and large objects with self-check query 16. Rehearse on a staging deployment that you restore from a recent production backup. If the rehearsal fails, contact support. |
| Write activity is high when the upgrade starts, or many transaction logs wait for archiving. | After read-only mode begins, the HA members and transaction-log archiving get only a few minutes to catch up. A backlog that passes the precheck can take longer to archive, and the upgrade then fails. | Keep the load low from the time that the upgrade starts until writes resume, and run `VACUUM` on recently loaded tables before the upgrade. Check replication lag by using the query in [Availability during an upgrade](#upgrading-in-place-availability), and check archiving by using self-check query 5. |
| Writes continue after read-only mode begins, for example from read-only overrides, maintenance commands, running `pg_cron` jobs, or enabled subscriptions on PostgreSQL 17. | The HA members and archiving might not catch up in time, causing the upgrade to fail. | Stop these writers before you submit the upgrade, as described in [Availability during an upgrade](#upgrading-in-place-availability). |
| A client prepares a transaction after the prechecks. `PREPARE TRANSACTION` still works in read-only mode. | The upgrade fails while writes are blocked. A transaction that is prepared shortly before PostgreSQL stops can cause the conversion  to fail. | Stop clients that use two-phase commit until the task shows **Completed**. Check with self-check query 4. |
| A replication client connects during the upgrade. | A slot that it creates is not kept, and the upgrade can fail or stall while the database is unavailable. | Keep replication clients disconnected until the task shows **Completed**. Check with self-check queries 1 and 2. |
| A continuous bulk load runs after writes resume. | The task fails after writes resume, and the deployment runs the target version until support completes the upgrade. | Avoid bulk loads until the task shows **Completed**. |
| The deployment's `synchronous_commit` setting is `on`. | A failed upgrade can leave every database read-only until support recovers the deployment. | Change it to `local`, as described in [Before you begin](#upgrading-considerations). |
| You upgrade from PostgreSQL 16 or earlier to 17 or later, and an index, a materialized view, or extended statistics use a function without a `search_path` setting. | After the upgrade, `ANALYZE`, `REINDEX`, `CLUSTER`, `VACUUM FULL`, and `REFRESH MATERIALIZED VIEW` can fail for these objects. | Check with self-check query 17. See [SQL functions in indexes, partition keys, and generated columns](#sql-function-wrappers). |
{: caption="Conditions that the prechecks do not detect" caption-side="bottom"}

Do not change schemas, extensions, databases, logical replication slots, or subscriptions between submitting the upgrade and task completion.

#### Checking readiness yourself
{: #upgrading-in-place-self-checks}

Run the following queries as the `admin` user with `psql` before you submit the upgrade. They work on PostgreSQL 14 through 18. Queries 1 through 7 and query 15 cover the whole deployment, so run them once in any database. Run the other queries in each database.

1. Replication connections. Expect one row for each HA member, which is the number of data members minus one, each with the state `streaming`. Any other row is a read-only replica or another replication client. A row named `pg_basebackup` means that the service is copying the database to an HA member or to a read-only replica. Wait until the copy finishes, and then run the query again.

    ```sql
    SELECT application_name, client_addr, state
    FROM pg_catalog.pg_stat_replication
    ORDER BY application_name;
    ```
    {: pre}

2. Logical replication slots. Expect no rows.

    ```sql
    SELECT slot_name, plugin, database, active
    FROM pg_catalog.pg_replication_slots
    WHERE slot_type = 'logical';
    ```
    {: pre}

3. Logical subscriptions. On PostgreSQL 16 and earlier, expect no rows. On PostgreSQL 17, expect `f` in the `subenabled` column of every row.

    ```sql
    SELECT d.datname AS database_name, s.subname, s.subenabled
    FROM pg_catalog.pg_subscription AS s
    JOIN pg_catalog.pg_database AS d ON d.oid = s.subdbid;
    ```
    {: pre}

4. Prepared transactions. Expect no rows.

    ```sql
    SELECT gid, owner, database, prepared
    FROM pg_catalog.pg_prepared_xacts;
    ```
    {: pre}

5. Transaction-log archiving. Expect `last_failed_time` to be empty or earlier than `last_archived_time`, and `waiting_files` to be 0 or close to 0. A backlog that passes the precheck can still be too large to archive after read-only mode begins.

    ```sql
    SELECT failed_count, last_failed_time, last_archived_time
    FROM pg_catalog.pg_stat_archiver;
    ```
    {: pre}

    ```sql
    SELECT count(*) AS waiting_files,
           pg_catalog.pg_size_pretty(count(*) * pg_catalog.pg_size_bytes(pg_catalog.current_setting('wal_segment_size'))) AS waiting_size
    FROM pg_catalog.pg_ls_archive_statusdir()
    WHERE name LIKE '%.ready';
    ```
    {: pre}

6. Database connection settings. Expect no rows. Drop a listed database whose `datconnlimit` is `-2`, because it is invalid. For any other listed database, allow connections or drop it. If the query lists `template0`, contact support.

    ```sql
    SELECT datname, datallowconn, datconnlimit
    FROM pg_catalog.pg_database
    WHERE (datname <> 'template0' AND (NOT datallowconn OR datconnlimit = -2))
       OR (datname = 'template0' AND datallowconn);
    ```
    {: pre}

7. Role and database settings. Make sure that every parameter in `setconfig` exists in the target version. Reset a parameter that the target version removed or renamed, and set its replacement after the upgrade. This also applies to extension-defined parameters that extensions, which contain a period in their names. Make sure that no role sets `default_transaction_read_only` to a false value, such as `off`, `false`, `no`, or `0`. Record each database that sets `default_transaction_read_only`, because the upgrade resets this setting. Self-check query 14 checks settings on functions.

    ```sql
    SELECT r.rolname AS role_name, d.datname AS database_name, s.setconfig
    FROM pg_catalog.pg_db_role_setting AS s
    LEFT JOIN pg_catalog.pg_roles AS r ON r.oid = s.setrole
    LEFT JOIN pg_catalog.pg_database AS d ON d.oid = s.setdatabase;
    ```
    {: pre}

8. Column data types that PostgreSQL cannot upgrade. Expect no rows. The query also lists columns of system composite types, and of domains, arrays, composite types, and ranges that are built on these types. The `base_type` column shows the type that blocks the upgrade. The `aclitem` type blocks only upgrades from PostgreSQL 14 or 15 to 16 or later.

    ```sql
    WITH RECURSIVE problem_types (type_oid, base_type) AS (
      SELECT t.oid, t.typname::text
      FROM pg_catalog.pg_type AS t
      WHERE t.typnamespace = 'pg_catalog'::pg_catalog.regnamespace
        AND t.typname IN ('regcollation', 'regconfig', 'regdictionary', 'regnamespace',
                          'regoper', 'regoperator', 'regproc', 'regprocedure', 'aclitem')
      UNION ALL
      SELECT t.oid, 'system composite type ' || t.oid::pg_catalog.regtype::text
      FROM pg_catalog.pg_type AS t
      LEFT JOIN pg_catalog.pg_namespace AS n ON n.oid = t.typnamespace
      WHERE t.typtype = 'c'
        AND (t.oid < 16384 OR n.nspname = 'information_schema')
      UNION ALL
      SELECT derived.type_oid, derived.base_type
      FROM (
        WITH found AS (SELECT type_oid, base_type FROM problem_types)
        SELECT t.oid AS type_oid, f.base_type
        FROM pg_catalog.pg_type AS t
        JOIN found AS f ON t.typtype = 'd' AND t.typbasetype = f.type_oid
        UNION ALL
        SELECT t.oid, f.base_type
        FROM pg_catalog.pg_type AS t
        JOIN found AS f ON t.typtype = 'b' AND t.typelem = f.type_oid
        UNION ALL
        SELECT t.oid, f.base_type
        FROM pg_catalog.pg_type AS t
        JOIN pg_catalog.pg_class AS c ON c.reltype = t.oid
        JOIN pg_catalog.pg_attribute AS a ON a.attrelid = c.oid AND NOT a.attisdropped
        JOIN found AS f ON a.atttypid = f.type_oid
        WHERE t.typtype = 'c'
        UNION ALL
        SELECT t.oid, f.base_type
        FROM pg_catalog.pg_type AS t
        JOIN pg_catalog.pg_range AS r ON r.rngtypid = t.oid
        JOIN found AS f ON r.rngsubtype = f.type_oid
        WHERE t.typtype = 'r'
      ) AS derived
    )
    SELECT DISTINCT n.nspname AS schema_name, c.relname AS relation_name, a.attname AS column_name,
           a.atttypid::pg_catalog.regtype AS data_type, p.base_type
    FROM pg_catalog.pg_attribute AS a
    JOIN pg_catalog.pg_class AS c ON c.oid = a.attrelid
    JOIN pg_catalog.pg_namespace AS n ON n.oid = c.relnamespace
    JOIN problem_types AS p ON p.type_oid = a.atttypid
    WHERE a.attnum > 0
      AND NOT a.attisdropped
      AND c.relkind IN ('r', 'm', 'i')
      AND n.nspname NOT IN ('pg_catalog', 'information_schema')
      AND n.nspname !~ '^pg_toast_temp_|^pg_temp_'
    ORDER BY 1, 2, 3;
    ```
    {: pre}

9. Inherited columns without a `NOT NULL` constraint that their parent table has. Expect no rows. To fix a listed column, the table owner runs `ALTER TABLE <table_name> ALTER COLUMN <column_name> SET NOT NULL;`.

    ```sql
    SELECT n.nspname AS schema_name, c.relname AS table_name, child.attname AS column_name
    FROM pg_catalog.pg_inherits AS i
    JOIN pg_catalog.pg_attribute AS child ON child.attrelid = i.inhrelid
    JOIN pg_catalog.pg_attribute AS parent
      ON parent.attrelid = i.inhparent AND parent.attname = child.attname
    JOIN pg_catalog.pg_class AS c ON c.oid = i.inhrelid
    JOIN pg_catalog.pg_namespace AS n ON n.oid = c.relnamespace
    WHERE parent.attnum > 0
      AND parent.attnotnull
      AND NOT child.attnotnull
      AND NOT child.attisdropped;
    ```
    {: pre}

10. Installed extensions. Complete the [extension preparation](#extensions). Then, confirm that every remaining extension is available in the target version, for example on a staging deployment of that version.

    ```sql
    SELECT extname, extversion
    FROM pg_catalog.pg_extension
    ORDER BY extname;
    ```
    {: pre}

11. `UNLOGGED` tables and sequences. Their contents do not survive the upgrade, and a listed table makes the service rebuild every HA member. Before you upgrade, see [`UNLOGGED` tables and sequences](#unlogged-objects).

    ```sql
    SELECT n.nspname AS schema_name, c.relname AS relation_name, c.relkind
    FROM pg_catalog.pg_class AS c
    JOIN pg_catalog.pg_namespace AS n ON n.oid = c.relnamespace
    WHERE c.relpersistence = 'u'
      AND c.relkind IN ('r', 'S');
    ```
    {: pre}

12. SQL functions that can make the conversion fail. Expect no rows. The query lists `LANGUAGE sql` functions with a string body that an index, a partial-index predicate, a partition key, a stored generated column, or extended statistics use. It follows calls through operators, domain check constraints, and functions with an SQL-standard body. For each listed function, see [SQL functions in indexes, partition keys, and generated columns](#sql-function-wrappers).

    ```sql
    WITH RECURSIVE uses (refclassid, refobjid, used_by) AS (
      SELECT d.refclassid, d.refobjid,
             CASE WHEN c.relkind = 'p' THEN 'partition key of ' ELSE 'index ' END
               || c.oid::pg_catalog.regclass::text
      FROM pg_catalog.pg_depend AS d
      JOIN pg_catalog.pg_class AS c
        ON d.classid = 'pg_catalog.pg_class'::pg_catalog.regclass
       AND c.oid = d.objid
      WHERE c.relkind IN ('i', 'I') OR (c.relkind = 'p' AND d.objsubid = 0)
      UNION
      SELECT d.refclassid, d.refobjid,
             'generated column ' || a.attname || ' of ' || a.attrelid::pg_catalog.regclass::text
      FROM pg_catalog.pg_depend AS d
      JOIN pg_catalog.pg_attrdef AS ad
        ON d.classid = 'pg_catalog.pg_attrdef'::pg_catalog.regclass
       AND ad.oid = d.objid
      JOIN pg_catalog.pg_attribute AS a
        ON a.attrelid = ad.adrelid
       AND a.attnum = ad.adnum
      WHERE a.attgenerated = 's'
      UNION
      SELECT d.refclassid, d.refobjid,
             'generated column ' || a.attname || ' of ' || a.attrelid::pg_catalog.regclass::text
      FROM pg_catalog.pg_depend AS d
      JOIN pg_catalog.pg_attribute AS a
        ON d.classid = 'pg_catalog.pg_class'::pg_catalog.regclass
       AND a.attrelid = d.objid
       AND a.attnum = d.objsubid
      WHERE a.attgenerated = 's'
      UNION
      SELECT d.refclassid, d.refobjid, 'statistics ' || s.stxname || ' on ' || s.stxrelid::pg_catalog.regclass::text
      FROM pg_catalog.pg_depend AS d
      JOIN pg_catalog.pg_statistic_ext AS s
        ON d.classid = 'pg_catalog.pg_statistic_ext'::pg_catalog.regclass
       AND s.oid = d.objid
    ), used (fn, used_by) AS (
      SELECT x.fn, u.used_by
      FROM uses AS u
      CROSS JOIN LATERAL (
        SELECT u.refobjid AS fn
        WHERE u.refclassid = 'pg_catalog.pg_proc'::pg_catalog.regclass
        UNION ALL
        SELECT o.oprcode::pg_catalog.oid
        FROM pg_catalog.pg_operator AS o
        WHERE u.refclassid = 'pg_catalog.pg_operator'::pg_catalog.regclass
          AND o.oid = u.refobjid
        UNION ALL
        SELECT dc.refobjid
        FROM pg_catalog.pg_constraint AS con
        JOIN pg_catalog.pg_depend AS dc
          ON dc.classid = 'pg_catalog.pg_constraint'::pg_catalog.regclass
         AND dc.objid = con.oid
         AND dc.refclassid = 'pg_catalog.pg_proc'::pg_catalog.regclass
        WHERE u.refclassid = 'pg_catalog.pg_type'::pg_catalog.regclass
          AND con.contypid = u.refobjid
      ) AS x
      UNION
      SELECT x.fn, u.used_by
      FROM used AS u
      JOIN pg_catalog.pg_proc AS p
        ON p.oid = u.fn
       AND p.prosqlbody IS NOT NULL
       AND p.proconfig IS NULL
       AND NOT p.prosecdef
      JOIN pg_catalog.pg_depend AS d
        ON d.classid = 'pg_catalog.pg_proc'::pg_catalog.regclass
       AND d.objid = u.fn
      CROSS JOIN LATERAL (
        SELECT d.refobjid AS fn
        WHERE d.refclassid = 'pg_catalog.pg_proc'::pg_catalog.regclass
        UNION ALL
        SELECT o.oprcode::pg_catalog.oid
        FROM pg_catalog.pg_operator AS o
        WHERE d.refclassid = 'pg_catalog.pg_operator'::pg_catalog.regclass
          AND o.oid = d.refobjid
      ) AS x
    )
    SELECT DISTINCT u.fn::pg_catalog.regprocedure AS function_name, u.used_by
    FROM used AS u
    JOIN pg_catalog.pg_proc AS p ON p.oid = u.fn
    JOIN pg_catalog.pg_language AS l ON l.oid = p.prolang
    WHERE l.lanname = 'sql'
      AND p.prosqlbody IS NULL
      AND p.proconfig IS NULL
      AND NOT p.prosecdef
      AND p.pronamespace <> 'pg_catalog'::pg_catalog.regnamespace
      AND NOT EXISTS (SELECT 1
                      FROM pg_catalog.pg_depend AS e
                      WHERE e.classid = 'pg_catalog.pg_proc'::pg_catalog.regclass
                        AND e.objid = p.oid
                        AND e.deptype = 'e')
    ORDER BY 1, 2;
    ```
    {: pre}

13. Indexes and stored generated columns that use `earthdistance` version 1.1, directly or through functions with an SQL-standard body. Expect no rows. For each listed object, see [`earthdistance`](#earthdistance).

    ```sql
    WITH RECURSIVE used (fn, via, used_by) AS (
      SELECT d.refobjid, d.refobjid, 'index ' || c.oid::pg_catalog.regclass::text
      FROM pg_catalog.pg_depend AS d
      JOIN pg_catalog.pg_class AS c
        ON d.classid = 'pg_catalog.pg_class'::pg_catalog.regclass
       AND c.oid = d.objid
       AND c.relkind IN ('i', 'I')
      WHERE d.refclassid = 'pg_catalog.pg_proc'::pg_catalog.regclass
      UNION
      SELECT d.refobjid, d.refobjid,
             'generated column ' || a.attname || ' of ' || a.attrelid::pg_catalog.regclass::text
      FROM pg_catalog.pg_depend AS d
      LEFT JOIN pg_catalog.pg_attrdef AS ad
        ON d.classid = 'pg_catalog.pg_attrdef'::pg_catalog.regclass
       AND ad.oid = d.objid
      JOIN pg_catalog.pg_attribute AS a
        ON a.attgenerated = 's'
       AND ((a.attrelid = ad.adrelid AND a.attnum = ad.adnum)
            OR (d.classid = 'pg_catalog.pg_class'::pg_catalog.regclass
                AND a.attrelid = d.objid
                AND a.attnum = d.objsubid))
      WHERE d.refclassid = 'pg_catalog.pg_proc'::pg_catalog.regclass
      UNION
      SELECT d.refobjid, u.via, u.used_by
      FROM used AS u
      JOIN pg_catalog.pg_proc AS p
        ON p.oid = u.fn
       AND p.prosqlbody IS NOT NULL
       AND p.proconfig IS NULL
      JOIN pg_catalog.pg_depend AS d
        ON d.classid = 'pg_catalog.pg_proc'::pg_catalog.regclass
       AND d.objid = u.fn
       AND d.refclassid = 'pg_catalog.pg_proc'::pg_catalog.regclass
    )
    SELECT e.extversion, u.via::pg_catalog.regprocedure AS used_function, u.used_by
    FROM used AS u
    JOIN pg_catalog.pg_depend AS m
      ON m.classid = 'pg_catalog.pg_proc'::pg_catalog.regclass
     AND m.objid = u.fn
     AND m.refclassid = 'pg_catalog.pg_extension'::pg_catalog.regclass
     AND m.deptype = 'e'
    JOIN pg_catalog.pg_extension AS e ON e.oid = m.refobjid
    JOIN pg_catalog.pg_proc AS p ON p.oid = u.fn
    JOIN pg_catalog.pg_language AS l ON l.oid = p.prolang
    WHERE e.extname = 'earthdistance'
      AND e.extversion IN ('1.0', '1.1')
      AND l.lanname = 'sql'
      AND p.proconfig IS NULL
    ORDER BY 2, 3;
    ```
    {: pre}

14. Views, materialized views, aggregates, and functions that use system catalog names or configuration parameters that a later PostgreSQL version removed. The `removed_in` column shows the first version without the name. Expect no rows with a `removed_in` version at or below your target version. A name can match text that is not a catalog reference, so review each listed object. See [Objects that use removed system names](#removed-system-objects).

    ```sql
    WITH removed (relation_name, identifier, removed_in) AS (
      VALUES (NULL, 'pg_is_in_backup', 15), (NULL, 'pg_backup_start_time', 15),
             (NULL, 'pg_start_backup', 15), (NULL, 'pg_stop_backup', 15),
             ('pg_database', 'datlastsysoid', 15),
             (NULL, 'force_parallel_mode', 16),
             ('pg_stat_bgwriter', 'checkpoints_timed', 17), ('pg_stat_bgwriter', 'checkpoints_req', 17),
             ('pg_stat_bgwriter', 'checkpoint_write_time', 17), ('pg_stat_bgwriter', 'checkpoint_sync_time', 17),
             ('pg_stat_bgwriter', 'buffers_checkpoint', 17), ('pg_stat_bgwriter', 'buffers_backend', 17),
             ('pg_stat_bgwriter', 'buffers_backend_fsync', 17),
             (NULL, 'pg_stat_get_bgwriter_timed_checkpoints', 17),
             (NULL, 'pg_stat_get_bgwriter_requested_checkpoints', 17),
             (NULL, 'pg_stat_get_checkpoint_write_time', 17), (NULL, 'pg_stat_get_checkpoint_sync_time', 17),
             (NULL, 'pg_stat_get_bgwriter_buf_written_checkpoints', 17),
             (NULL, 'pg_stat_get_buf_written_backend', 17), (NULL, 'pg_stat_get_buf_fsync_backend', 17),
             ('pg_stat_progress_vacuum', 'max_dead_tuples', 17),
             ('pg_stat_progress_vacuum', 'num_dead_tuples', 17),
             ('pg_database', 'daticulocale', 17), ('pg_collation', 'colliculocale', 17),
             ('element_types', 'domain_default', 17),
             (NULL, 'interval_accum', 17), (NULL, 'interval_accum_inv', 17), (NULL, 'interval_combine', 17),
             ('pg_stat_wal', 'wal_write', 18), ('pg_stat_wal', 'wal_sync', 18),
             ('pg_stat_wal', 'wal_write_time', 18), ('pg_stat_wal', 'wal_sync_time', 18),
             ('pg_stat_io', 'op_bytes', 18), ('pg_backend_memory_contexts', 'parent', 18),
             ('pg_attribute', 'attcacheoff', 18)
    ), definitions (object_name, definition) AS (
      SELECT 'view ' || c.oid::pg_catalog.regclass::text, pg_catalog.pg_get_viewdef(c.oid)
      FROM pg_catalog.pg_class AS c
      WHERE c.relkind IN ('v', 'm')
        AND c.relnamespace NOT IN ('pg_catalog'::pg_catalog.regnamespace,
                                   'information_schema'::pg_catalog.regnamespace)
      UNION ALL
      SELECT 'function ' || p.oid::pg_catalog.regprocedure::text,
             pg_catalog.concat_ws(' ', pg_catalog.pg_get_function_sqlbody(p.oid),
                                  pg_catalog.array_to_string(p.proconfig, ' '))
      FROM pg_catalog.pg_proc AS p
      WHERE (p.prosqlbody IS NOT NULL OR p.proconfig IS NOT NULL)
        AND p.pronamespace NOT IN ('pg_catalog'::pg_catalog.regnamespace,
                                   'information_schema'::pg_catalog.regnamespace)
      UNION ALL
      SELECT 'aggregate ' || a.aggfnoid::pg_catalog.regprocedure::text,
             pg_catalog.concat_ws(' ', a.aggtransfn, a.aggfinalfn, a.aggcombinefn,
                                  a.aggmtransfn, a.aggminvtransfn, a.aggmfinalfn)
      FROM pg_catalog.pg_aggregate AS a
      JOIN pg_catalog.pg_proc AS p ON p.oid = a.aggfnoid
      WHERE p.pronamespace NOT IN ('pg_catalog'::pg_catalog.regnamespace,
                                   'information_schema'::pg_catalog.regnamespace)
    )
    SELECT d.object_name, r.identifier, r.removed_in
    FROM definitions AS d
    JOIN removed AS r
      ON d.definition ~ ('\m' || r.identifier || '\M')
     AND (r.relation_name IS NULL OR d.definition ~ ('\m' || r.relation_name || '\M'))
    ORDER BY 3, 1, 2;
    ```
    {: pre}

15. Database names that the upgrade cannot use. Expect no rows. To fix a listed database, its owner must connect to another database and run `ALTER DATABASE "<database_name>" RENAME TO <new_name>;` while no sessions are connected to to the database being renamed. The owner must also have the `CREATEDB` attribute. Then, update any applications that connect to the database.

    ```sql
    SELECT datname
    FROM pg_catalog.pg_database
    WHERE pg_catalog.strpos(datname, '''') > 0
       OR pg_catalog.strpos(datname, pg_catalog.chr(92)) > 0
       OR pg_catalog.strpos(datname, '=') > 0
       OR pg_catalog.strpos(datname, pg_catalog.chr(10)) > 0
       OR pg_catalog.strpos(datname, pg_catalog.chr(13)) > 0
    ORDER BY datname;
    ```
    {: pre}

16. Relations that the upgrade locks, large objects, and the deployment's lock capacity. Add up `locked_relations` for the four databases with the highest values. On PostgreSQL 14 and 15, `locked_relations` includes views and materialized views, because the upgrade also locks them. If the total exceeds `lock_capacity`, increase [`max_locks_per_transaction`](/docs/databases-for-postgresql?topic=databases-for-postgresql-changing-configuration#gen-settings) until `lock_capacity` exceeds the total. The change restarts the database, so make it before you submit the upgrade. Many tables or large objects also lengthen the conversion, so rehearse the upgrade with your real schema and data.

    ```sql
    SELECT pg_catalog.current_database() AS database_name,
           (SELECT pg_catalog.count(*)
            FROM pg_catalog.pg_class AS c
            WHERE (c.relkind IN ('r', 'p', 'S')
                   OR (c.relkind IN ('v', 'm')
                       AND pg_catalog.current_setting('server_version_num')::integer < 160000))
              AND c.relnamespace NOT IN ('pg_catalog'::pg_catalog.regnamespace,
                                         'information_schema'::pg_catalog.regnamespace)) AS locked_relations,
           (SELECT pg_catalog.count(*)
            FROM pg_catalog.pg_largeobject_metadata) AS large_objects,
           pg_catalog.current_setting('max_locks_per_transaction')::integer
             * (pg_catalog.current_setting('max_connections')::integer
                + pg_catalog.current_setting('max_prepared_transactions')::integer) AS lock_capacity;
    ```
    {: pre}

17. Functions without a `search_path` setting that indexes, materialized views, and extended statistics use. The query follows calls through operators, domain check constraints, and functions with an SQL-standard body. Run this query if you upgrade from PostgreSQL 16 or earlier to PostgreSQL 17 or later. A listed function whose body names objects outside `pg_catalog` without their schema makes maintenance commands fail after the upgrade. A function is unaffected only if its body, and every function that it calls, names every object outside `pg_catalog` with its schema. For each other listed function, its owner adds a `search_path` setting, as described in [SQL functions in indexes, partition keys, and generated columns](#sql-function-wrappers).

    ```sql
    WITH RECURSIVE refreshed (relid, used_by) AS (
      SELECT c.oid, c.oid::pg_catalog.regclass
      FROM pg_catalog.pg_class AS c
      WHERE c.relkind = 'm'
      UNION
      SELECT v.oid, f.used_by
      FROM refreshed AS f
      JOIN pg_catalog.pg_rewrite AS r ON r.ev_class = f.relid
      JOIN pg_catalog.pg_depend AS d
        ON d.classid = 'pg_catalog.pg_rewrite'::pg_catalog.regclass
       AND d.objid = r.oid
       AND d.refclassid = 'pg_catalog.pg_class'::pg_catalog.regclass
      JOIN pg_catalog.pg_class AS v ON v.oid = d.refobjid AND v.relkind = 'v'
    ), refs (refclassid, refobjid, used_by) AS (
      SELECT d.refclassid, d.refobjid, c.oid::pg_catalog.regclass
      FROM pg_catalog.pg_depend AS d
      JOIN pg_catalog.pg_class AS c ON c.oid = d.objid AND c.relkind IN ('i', 'I')
      WHERE d.classid = 'pg_catalog.pg_class'::pg_catalog.regclass
      UNION
      SELECT d.refclassid, d.refobjid, f.used_by
      FROM refreshed AS f
      JOIN pg_catalog.pg_rewrite AS r ON r.ev_class = f.relid
      JOIN pg_catalog.pg_depend AS d
        ON d.classid = 'pg_catalog.pg_rewrite'::pg_catalog.regclass
       AND d.objid = r.oid
      UNION
      SELECT d.refclassid, d.refobjid, s.stxrelid::pg_catalog.regclass
      FROM pg_catalog.pg_depend AS d
      JOIN pg_catalog.pg_statistic_ext AS s ON s.oid = d.objid
      WHERE d.classid = 'pg_catalog.pg_statistic_ext'::pg_catalog.regclass
    ), used (fn, used_by) AS (
      SELECT x.fn, r.used_by
      FROM refs AS r
      CROSS JOIN LATERAL (
        SELECT r.refobjid AS fn
        WHERE r.refclassid = 'pg_catalog.pg_proc'::pg_catalog.regclass
        UNION ALL
        SELECT o.oprcode::pg_catalog.oid
        FROM pg_catalog.pg_operator AS o
        WHERE r.refclassid = 'pg_catalog.pg_operator'::pg_catalog.regclass
          AND o.oid = r.refobjid
        UNION ALL
        SELECT dc.refobjid
        FROM pg_catalog.pg_constraint AS con
        JOIN pg_catalog.pg_depend AS dc
          ON dc.classid = 'pg_catalog.pg_constraint'::pg_catalog.regclass
         AND dc.objid = con.oid
         AND dc.refclassid = 'pg_catalog.pg_proc'::pg_catalog.regclass
        WHERE r.refclassid = 'pg_catalog.pg_type'::pg_catalog.regclass
          AND con.contypid = r.refobjid
      ) AS x
      UNION
      SELECT x.fn, u.used_by
      FROM used AS u
      JOIN pg_catalog.pg_proc AS p
        ON p.oid = u.fn
       AND p.prosqlbody IS NOT NULL
       AND NOT EXISTS (SELECT 1
                       FROM pg_catalog.unnest(p.proconfig) AS setting
                       WHERE setting LIKE 'search\_path=%')
      JOIN pg_catalog.pg_depend AS d
        ON d.classid = 'pg_catalog.pg_proc'::pg_catalog.regclass
       AND d.objid = u.fn
      CROSS JOIN LATERAL (
        SELECT d.refobjid AS fn
        WHERE d.refclassid = 'pg_catalog.pg_proc'::pg_catalog.regclass
        UNION ALL
        SELECT o.oprcode::pg_catalog.oid
        FROM pg_catalog.pg_operator AS o
        WHERE d.refclassid = 'pg_catalog.pg_operator'::pg_catalog.regclass
          AND o.oid = d.refobjid
      ) AS x
    )
    SELECT DISTINCT p.oid::pg_catalog.regprocedure AS function_name, u.used_by
    FROM used AS u
    JOIN pg_catalog.pg_proc AS p ON p.oid = u.fn
    JOIN pg_catalog.pg_language AS l ON l.oid = p.prolang
    WHERE l.lanname NOT IN ('c', 'internal')
      AND p.prosqlbody IS NULL
      AND p.pronamespace NOT IN ('pg_catalog'::pg_catalog.regnamespace,
                                 'information_schema'::pg_catalog.regnamespace)
      AND NOT EXISTS (SELECT 1
                      FROM pg_catalog.unnest(p.proconfig) AS setting
                      WHERE setting LIKE 'search\_path=%')
      AND NOT EXISTS (SELECT 1
                      FROM pg_catalog.pg_depend AS e
                      WHERE e.classid = 'pg_catalog.pg_proc'::pg_catalog.regclass
                        AND e.objid = p.oid
                        AND e.deptype = 'e')
    ORDER BY 1, 2;
    ```
    {: pre}

Passing these checks does not guarantee that the upgrade succeeds or that your applications are compatible with the target version. Only a rehearsal on a staging deployment runs the whole conversion.

### Planning the upgrade window
{: #upgrading-in-place-window}

Use your staging rehearsal to estimate a maintenance window. The conversion time depends mainly on the number of databases and objects. Rebuilding an HA member depends on the size of the database. Workload and transaction-log volume affect the checks and the catch-up. Allow time for the whole upgrade task, not only the time that writes are blocked.

Start expiration sets the latest time that a queued upgrade can start. It defaults to 5 minutes after the request, and you can set it up to 24 hours ahead. Running operations, such as a backup, and earlier queued requests delay the start. If the upgrade does not start before its expiration time, it does not run, and its task shows **Expired**. Expiration does not stop an upgrade that has started. For example, a queued request expires at 22:30 UTC. A task that starts at 22:25 UTC can continue past that time.

While the upgrade runs, other requests for the deployment wait until it finishes. These include manual and automatic scaling, configuration changes, changes to users and passwords, changes to allowed IP addresses, and backups. Do not delete the deployment while the upgrade task is queued or running.

Do not create read-only replicas until the task completes. The service does not block such requests. If you create one, the upgrade can fail its prechecks, or the new replica cannot replicate from the upgraded deployment. Delete such a replica, and create a new one after the task completes.

### Upgrading in the UI
{: #upgrading-in-place-ui}
{: ui}

1. Create and verify an [on-demand backup](/docs/cloud-databases?topic=cloud-databases-dashboard-backups). The upgrade does not take one.
2. On the deployment's **Overview** page, click **Upgrade major version**.
3. In the **Guidance** step, review the guidance, select **I have read the guidance and am ready to proceed with the upgrade**, and click **Next**. To learn when reads and writes are unavailable, see [Availability during an upgrade](#upgrading-in-place-availability).
4. In **Upgrade to version**, select the target version. If more than one target is available, the list selects the newest one by default. The **Read-replicas** notice in this step is out of date. The upgrade fails its prechecks while any read-only replica is associated with the deployment, so promote or delete every replica before you upgrade.
5. In **Expiration for starting upgrade**, select how long the request can wait to start. The default is 5 minutes.
6. Click **Upgrade**. While the request waits or runs, the **Version** row on the **Overview** page shows **Upgrade queued** or **Upgrade in progress**.
7. Monitor the task in **Recent tasks**. As soon as writes resume, refresh statistics as described in [After the upgrade](#upgrading-in-place-after). When the task shows **Completed**, complete the remaining steps in that section. If it shows **Failed**, see [Troubleshooting](#upgrading-in-place-troubleshooting). If it shows **Expired**, the upgrade did not start, and you can submit it again with a later expiration.

### Upgrading through the API
{: #upgrading-in-place-api}
{: api}

Send a `PATCH` request to the `/deployments/{id}/version` endpoint. Set `version` to the target major version, and set the start expiration with the `expiration_datetime` query parameter:

```sh
curl -X PATCH \
  'https://api.{region}.databases.cloud.ibm.com/v5/ibm/deployments/{id}/version?expiration_datetime=<EXPIRATION_TIME>' \
  -H 'Authorization: Bearer <IAM_TOKEN>' \
  -H 'Content-Type: application/json' \
  -d '{"version": "18"}'
```
{: pre}

Replace `<EXPIRATION_TIME>` with a UTC time in ISO 8601 format, such as `2026-11-03T22:30:00Z`, from 5 minutes to 24 hours after the request. If you omit `expiration_datetime`, the request expires if it does not start within 5 minutes.

A successful request returns a task. An accepted request is not a completed upgrade. Use the task ID with [Get information about a task](/apidocs/cloud-databases-api/cloud-databases-api-v5#gettask) to monitor the upgrade. To list the available targets, use [Discover capability information from a deployment](/apidocs/cloud-databases-api/cloud-databases-api-v5#getdeploymentcapability) with the `versions` capability. For all parameters, see the [API reference](/apidocs/cloud-databases-api/cloud-databases-api-v5#setdatabaseinplaceversionupgrade).

### Upgrading through the CLI
{: #upgrading-in-place-cli}
{: cli}

Use version 0.20.0 or later of the {{site.data.keyword.databases-for}} CLI plug-in. Set the start expiration with `--expire-in` or `--expire-at`, from 5 minutes to 24 hours after the request. The default is 5 minutes.

```sh
ibmcloud cdb deployment-version-upgrade <NAME|CRN> <TARGET_VERSION> --expire-in 1h --nowait
```
{: pre}

Without `--nowait`, the command waits until the task completes or fails. It does not return if the upgrade expires before it starts, so use `--nowait` in scripts. With `--nowait`, the command returns after the service accepts the request. Monitor the task with [`deployment-tasks-list`](/docs/cli?topic=cli-cdb-reference#deployment-tasks-list):

```sh
ibmcloud cdb deployment-tasks-list <NAME|CRN>
```
{: pre}

For all parameters, run `ibmcloud cdb deployment-version-upgrade --help`.

### Upgrading through Terraform
{: #upgrading-in-place-terraform}
{: terraform}

Use {{site.data.keyword.cloud}} Terraform provider version 1.79.2 or later. Set `version` to an available target, review the plan, and apply it. The plan fails if the target is not available for an in-place upgrade.

Set the resource's update timeout long enough for the whole upgrade, and to at least 5 minutes. The provider also uses this timeout, up to 24 hours, as the start expiration. A queued upgrade can therefore start at any time until the timeout elapses. Apply the change when no backup or other operation is running or queued, so that the upgrade starts immediately. If the apply times out after the upgrade starts, the upgrade continues. If the upgrade is still queued when the timeout elapses, it expires and does not run. Wait for the task to finish before you run `terraform plan` or `terraform apply` again.

If the upgrade task fails, complete [Troubleshooting](#upgrading-in-place-troubleshooting) before you run `terraform apply` again. Terraform submits the upgrade again whenever the deployment still reports the source version.

After an upgrade outside Terraform, set `version` to the new version, or remove `version` from the configuration. Upgrades outside Terraform include upgrades in the UI, CLI, or API, and a [forced upgrade](#forced_upgrade). Until you update the configuration, every `terraform plan` and `terraform apply` that includes the deployment fails, preventing changes to other resources in the same configuration. The error is similar to `Version 14 is not a valid upgrade version` or `No available upgrade versions for version 18`.

The `version_upgrade_skip_backup` argument is not supported for PostgreSQL. For more information, see the [database resource reference](https://registry.terraform.io/providers/IBM-Cloud/ibm/latest/docs/resources/database){: external}.

### After the upgrade
{: #upgrading-in-place-after}

As soon as writes resume, make sure that optimizer statistics are complete. Do not wait for the task to complete. Writes have resumed when `SHOW default_transaction_read_only;` returns `off` in a new session. In each database, the following query lists your tables and materialized views that have no statistics. Run `ANALYZE <table_name>;` for each listed relation. If your user does not have permission to analyze a relation, run the command as the relation owner. Analyzing a partitioned table also analyzes its partitions.

```sql
SELECT c.oid::pg_catalog.regclass AS relation_name
FROM pg_catalog.pg_class AS c
WHERE c.relkind IN ('r', 'm', 'p')
  AND c.reltuples < 0
  AND NOT c.relispartition
  AND c.relpersistence <> 't'
  AND c.relnamespace NOT IN ('pg_catalog'::pg_catalog.regnamespace,
                             'information_schema'::pg_catalog.regnamespace)
  AND NOT EXISTS (SELECT 1
                  FROM pg_catalog.pg_depend AS e
                  WHERE e.classid = 'pg_catalog.pg_class'::pg_catalog.regclass
                    AND e.objid = c.oid
                    AND e.deptype = 'e');
```
{: pre}

On PostgreSQL 17 and later, `ANALYZE` can fail with an error such as `function ... does not exist`. In that case, fix the function that self-check query 17 lists, and run `ANALYZE` again. A database-wide `ANALYZE` that stops with an error does not analyze the remaining tables. On PostgreSQL 18, also analyze each table that has statistics objects, as listed by the `psql` command `\dX`, and each table that contains PostGIS columns, because the upgrade does not transfer these statistics.

After the task shows **Completed**:

1. Confirm that the deployment's version in the console or API shows the target version, and that `SHOW server_version;` returns it. Run [self-check query 1](#upgrading-in-place-self-checks), and confirm that every HA member is streaming. Also confirm that writes resumed in every database: the following query returns no rows. If an HA member is missing, a row named `pg_basebackup` remains, or the query lists a database, contact support.

    ```sql
    SELECT d.datname AS database_name, s.setconfig
    FROM pg_catalog.pg_db_role_setting AS s
    JOIN pg_catalog.pg_database AS d ON d.oid = s.setdatabase
    WHERE s.setrole = 0
      AND d.datallowconn
      AND NOT d.datistemplate
      AND EXISTS (SELECT 1
                  FROM pg_catalog.unnest(s.setconfig) AS c(setting)
                  WHERE c.setting LIKE 'default_transaction_read_only=%');
    ```
    {: pre}

2. Update extensions. The upgrade keeps each extension at its installed version. In each database, list the extensions that have a newer version, and update each one:

    ```sql
    SELECT name, installed_version, default_version
    FROM pg_catalog.pg_available_extensions
    WHERE installed_version <> default_version;
    ```
    {: pre}

    ```sql
    ALTER EXTENSION <extension_name> UPDATE;
    ```
    {: pre}

    If you upgraded from PostgreSQL 14, 15, or 16 to PostgreSQL 17 or later, updating `pg_stat_statements` renames its `blk_read_time` and `blk_write_time` columns to `shared_blk_read_time` and `shared_blk_write_time`. Update the queries that read these columns.

3. Restore extensions and masking as described in [preparation](#upgrading-reqs). Reload the `UNLOGGED` tables from the contents that you exported. Set each `UNLOGGED` sequence past the values in use, for example with `SELECT setval('<sequence_name>', (SELECT max(<column_name>) FROM <table_name>));`. Recreate the logical replication slots, subscriptions, and read-only replicas that you removed, and enable the subscriptions that you disabled. Reconnect consumers, and resynchronize data before you resume dependent applications.
4. If you set `default_transaction_read_only` for a database, set it again. The upgrade resets this setting on every database, so those databases accept writes again.
5. Confirm that the [post-upgrade backup](#upgrading-in-place-backups) completed. It copies the whole database, so it can take several hours. If you need point-in-time recovery for changes made after the upgrade, keep application writes paused until then.
6. If you upgraded from PostgreSQL 14 or 15 to PostgreSQL 16 or later, run the role query in [Role privilege issues during version upgrades](#admin_user_issues). Grant the listed roles before you resume jobs that manage roles.
7. If Terraform manages the deployment and you upgraded outside Terraform, set `version` to the new version or remove it, as described in [Upgrading through Terraform](#upgrading-in-place-terraform).
8. Resume the application writes and scheduled jobs that you paused. Restore the settings that you changed for the upgrade, such as `synchronous_commit`.
9. Rebuild objects that depend on Unicode data, because newer PostgreSQL versions use newer Unicode data:

    - Unless you upgraded from PostgreSQL 16 to PostgreSQL 17, check indexes, materialized views, check constraints, and partition keys whose expressions use `normalize()` or `is_normalized()`. Only text that contains characters that are new in the later Unicode version is affected.
    - If you upgraded from PostgreSQL 17 to PostgreSQL 18, also check objects that use `unicode_assigned()`. Also check objects that apply regular expressions, `ILIKE`, `lower()`, `upper()`, or `initcap()` to text in the built-in `C.UTF-8` locale. The `pg_c_utf8` collation and databases that use the `builtin` locale provider with `C.UTF-8` use this locale.
    - If you upgraded to PostgreSQL 18 and a database uses the `icu` or `builtin` locale provider, rebuild its full-text search and `pg_trgm` indexes.

    Run `REINDEX INDEX <index_name>;` for each affected index and `REFRESH MATERIALIZED VIEW <view_name>;` for each affected materialized view. Make sure that rows still satisfy affected check constraints and are in the correct partitions.

10. Compare application behavior and performance with your staging results.

### Troubleshooting
{: #upgrading-in-place-troubleshooting}

If the service rejects a request, the error appears immediately. The UI usually shows **Upgrade failed** with the reason, the API returns the error message, and the CLI prints the error with a hint. Correct the request, and submit it again.

If the task shows **Expired**, the upgrade did not start, and the deployment is unchanged. Submit the upgrade again when no other operation is running or queued, or set a later start expiration.

A task that later shows **Failed** does not identify the failed check or step, and the UI does not show the cause. Depending on the failed step, the deployment can be in one of these states:

- **On the source version**: The deployment runs the source version and accepts writes. A failed precheck leaves the deployment in this state, and the service restores write access after most failures during read-only mode. A failed attempt can still have lasting effects. It can remove `old_snapshot`, empty `UNLOGGED` tables, reset `UNLOGGED` sequences, and leave a gap in PITR coverage. It can also reset database-level `default_transaction_read_only` settings, so those databases accept writes again. If you set `default_transaction_read_only` to `on` for any database, a failed attempt also ends all client sessions, even when a precheck fails.
- **On the target version**: The deployment runs the target version, but the upgrade is incomplete until support completes it. The console and API can still show the source version. Until then, the HA members might not run, and backups and transaction-log archiving can fail. If self-check query 1 or 5 shows a problem, pause application writes until support completes the upgrade.
- **Impaired**: The deployment remains read-only or unavailable, or runs without its HA members, until support recovers it. A failure in the conversion or restart phase can leave the HA members stopped until support restarts them, even after writes resume.

To identify the state, connect as `admin` and run `SHOW server_version;`. In each database that you use, run `SHOW default_transaction_read_only;`. Run [self-check query 1](#upgrading-in-place-self-checks) to confirm that every HA member is streaming. If `admin` cannot connect after the task fails, for example because of an authentication error, contact support. The upgrade does not change your credentials.

If the deployment accepts writes, runs the source version, and every HA member is streaming, review the [prechecks](#upgrading-in-place-prechecks) and run the [self-check queries](#upgrading-in-place-self-checks). Resolve every unmet item, take an on-demand backup, and then submit the upgrade again. In every other case, or if the next attempt also fails, [contact support](https://cloud.ibm.com/unifiedsupport/supportcenter) before you retry or make other changes. Include:

- The deployment CRN and the failed task ID. Record the task ID when the task fails, because task lists show only recent tasks.
- The source and requested target PostgreSQL versions, and the output of `SHOW server_version;`.
- The approximate failure time and time zone.
- Whether applications can connect, read, and write.
- The output of the self-check queries.
- The CRN of each related read-only replica, including any recently promoted or deleted replica.

If the request fails with an internal error, contact support. In the UI, this error shows **Upgrade failed** with the message **Failed to create the in-place upgrade task**. The UI also displays this message for other rejected requests when it cannot display the actual reason. To see the reason, submit the request by using the CLI or API. Also contact support if the task stays in progress much longer than in your rehearsal. After a failure, the task can stay in progress (status `running` in the CLI and API) while the service attempts recovery.

Remove only the objects that the preparation steps and the self-check queries list. Do not drop other objects to get past a check. Contact support instead.

## Upgrading from a read-only replica
{: #upgrading-replica}

[Create a read-only replica](/docs/databases-for-postgresql?topic=databases-for-postgresql-read-only-replicas) from the source deployment and wait for the replica to synchronize. Then, promote and upgrade the replica by using the [`/remotes/promotion` endpoint](/apidocs/cloud-databases-api/cloud-databases-api-v5#promotereadonlyreplica). Set `version` to a target that the replica lists in its capabilities:

```sh
curl -X POST \
  https://api.{region}.databases.cloud.ibm.com/v5/ibm/deployments/{id}/remotes/promotion \
  -H 'Authorization: Bearer <IAM_TOKEN>' \
  -H 'Content-Type: application/json' \
  -d '{
    "promotion": {
        "version": "18",
        "skip_initial_backup": false
    }
}'
```
{: pre}

Set `skip_initial_backup` to `true` only if you want to skip the initial backup that promotion creates. This setting can shorten the promotion task, but recovery from the promoted deployment then requires a later successful scheduled or on-demand backup.

If a promotion with a version upgrade fails, the service disables the promoted deployment. Contact support. Rehearse a promotion with a version upgrade on a replica of a staging deployment before you rely on it.

## Upgrading by restoring a backup
{: #backup-restore}

[Restore a backup](/docs/cloud-databases?topic=cloud-databases-dashboard-backups&interface=ui#restore-backup-ui) to a deployment that uses a supported target version. If you restore a backup by using the CLI or API and do not specify the resource allocation parameters, the new deployment uses the resource allocations that the source deployment had when the backup was created. In the UI, the restore page starts with the source deployment's current allocations, which you can change.

### Restoring a backup in the UI
{: #upgrading-ui}
{: ui}

From **Backups and restore** in the deployment dashboard, click **Restore backup** for the backup that you want to use. On the page that opens, select a supported target version in **Database version**, and configure the options for the new deployment. Then, click **Restore backup**.

### Restoring a backup through the CLI
{: #upgrading-cli}
{: cli}

Create the deployment with the target version and backup ID in the `-p` JSON argument:

```sh
ibmcloud resource service-instance-create example-upgrade databases-for-postgresql standard us-south \
  -p '{
  "backup_id": "crn:v1:bluemix:public:databases-for-postgresql:us-south:a/54e8ffe85dcedf470db5b5ee6ac4a8d8:1b8f53db-fc2d-4e24-8470-f82b15c71717:backup:06392e97-df90-46d8-98e8-cb67e9e0a8e6",
  "version": "18"
}' \
  --service-endpoints "public"
```
{: pre}

### Restoring a backup through the API
{: #upgrading-api}
{: api}

Use the [Resource controller API](/docs/databases-for-postgresql?topic=databases-for-postgresql-provisioning&interface=api#provision-controller-api) to restore a backup into a new deployment that runs the target version. Specify the deployment name, location, resource group, plan, backup ID, and target version:

```sh
curl -X POST \
  https://resource-controller.cloud.ibm.com/v2/resource_instances \
  -H 'Authorization: Bearer <IAM_TOKEN>' \
  -H 'Content-Type: application/json' \
  -d '{
    "name": "my-instance",
    "target": "bluemix-us-south",
    "resource_group": "5g9f447903254bb58972a2f3f5a4c711",
    "resource_plan_id": "databases-for-postgresql-standard",
    "parameters": {
      "backup_id": "crn:v1:bluemix:public:databases-for-postgresql:us-south:a/54e8ffe85dcedf470db5b5ee6ac4a8d8:1b8f53db-fc2d-4e24-8470-f82b15c71717:backup:06392e97-df90-46d8-98e8-cb67e9e0a8e6",
      "version": "18"
    }
  }'
```
{: pre}

## Forced upgrade
{: #forced_upgrade}

After the end-of-life date, all active {{site.data.keyword.databases-for-postgresql}} deployments that run the deprecated version are automatically upgraded to the next supported version.
{: .note}

A forced upgrade is an IPMVU, so complete the [IPMVU preparation](#upgrading-in-place) before the end-of-life date.

**Upgrade before the end-of-life date to avoid the following risks:**

- No SLAs are provided for this type of forced upgrade.
- You might experience some data loss.
- Your application might experience prolonged downtime.
- Your application might stop working if it is incompatible with the new version.
- You cannot control the timing of when this upgrade will happen for your deployment.
- There is no rollback process for this forced upgrade.
- If Terraform manages the deployment and sets `version`, every `terraform plan` fails after the forced upgrade until you update `version`.

For the end-of-life dates, see the [version policy page](/docs/cloud-databases?topic=cloud-databases-versioning-policy).

## Role privilege issues during version upgrades
{: #admin_user_issues}

In PostgreSQL 16 and later, a role with the `CREATEROLE` attribute, such as `admin`, needs the `ADMIN OPTION` on another role to manage it. Managing a role includes changing its password or attributes, dropping it, and granting its membership. An upgrade from PostgreSQL 14 or 15 to 16 or later does not give `admin` this option on roles that existed before the upgrade. Jobs that manage these roles as `admin`, such as password rotation, then fail. For more information, see the [PostgreSQL 16 release notes](https://www.postgresql.org/docs/16/release-16.html){: external}, [role attributes](https://www.postgresql.org/docs/16/role-attributes.html){: external} and [role grants](https://www.postgresql.org/docs/16/sql-grant.html){: external}.

For example, a password change fails with:

```text
ERROR:  permission denied to alter role
DETAIL:  To change another role's password, the current user must have the CREATEROLE attribute and the ADMIN option on the role.
```
{: pre}

Before you upgrade, run the following query as `admin`. It lists the roles that `admin` cannot manage after the upgrade, except users that you create in the UI or with the CLI or API. On PostgreSQL 16 and later, `admin` cannot change those users either, so manage them in the UI or with the CLI or API.

```sql
SELECT r.rolname AS role_name,
       pg_catalog.pg_has_role(r.oid, 'admin'::name, 'MEMBER') AS member_of_admin
FROM pg_catalog.pg_roles AS r
WHERE NOT r.rolsuper
  AND NOT r.rolreplication
  AND r.rolname <> 'admin'
  AND r.rolname !~ '^(pg_|ibm-)'
  AND NOT EXISTS (
      SELECT 1
      FROM pg_catalog.pg_auth_members AS m
      JOIN pg_catalog.pg_roles AS b ON b.oid = m.roleid
      WHERE m.member = r.oid
        AND b.rolname IN ('ibm-cloud-base-user', 'ibm-cloud-base-user-ro'))
  AND NOT pg_catalog.pg_has_role(r.oid, 'MEMBER WITH ADMIN OPTION')
ORDER BY 1;
```
{: pre}

To keep managing a listed role, grant it to `admin` before you upgrade:

```sql
GRANT <role_name> TO admin WITH ADMIN OPTION;
```
{: pre}

This grant fails for a role whose `member_of_admin` value is `t`, because that role is a member of `admin`. After the upgrade, `admin` cannot manage such a role.

After an upgrade to PostgreSQL 16 or later, run the query again. For each listed role whose `member_of_admin` value is `f`, run the following helper while connected to `ibmclouddb` or any database except `postgres`. Do not include other roles, because one failed grant cancels the whole call. You can run the helper safely more than once:

```sql
SELECT grant_admin_option_to_roles('role1', 'role2', 'role3');
```
{: pre}

Both grants also make `admin` a member of the role, so `admin` can use the role's privileges.

## Changelog for major PostgreSQL versions
{: #changelog-postgres}

- [PostgreSQL 15](https://www.postgresql.org/docs/release/15.0/){: external}
- [PostgreSQL 16](https://www.postgresql.org/docs/release/16.0/){: external}
- [PostgreSQL 17](https://www.postgresql.org/docs/release/17.0/){: external}
- [PostgreSQL 18](https://www.postgresql.org/docs/release/18.0/){: external}
