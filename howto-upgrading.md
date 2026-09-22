---
copyright:
  years: 2020, 2026
lastupdated: "2026-09-22"

keywords: postgresql, databases, upgrading, major versions, postgresql new deployment, postgresql database version, postgresql major version

subcollection: databases-for-postgresql

---

{{site.data.keyword.attribute-definition-list}}

# Upgrading to a new major version
{: #upgrading}

Choose an upgrade path:

- **[In-place major version upgrade (IPMVU)](#upgrading-in-place):** Upgrade the existing deployment and retain its connection strings.
- **[Restore from backup](#backup-restore):** Create a deployment on a supported target version from an existing backup.
- **[Upgrade a read-only replica](#upgrading-replica):** Replicate data to a separate deployment, then promote and upgrade it.

For the last two paths, validate the new deployment before switching your application's connection details. Promotion ends replication from the source.

Find versions available for new deployments in the [catalog](https://cloud.ibm.com/databases/databases-for-postgresql/create), with [`ibmcloud cdb deployables-show`](/docs/cli?topic=cli-cdb-reference#deployables-show), or through the [`/deployables` API](/apidocs/cloud-databases-api/cloud-databases-api-v5#listdeployables).

In API examples, replace `{id}` with the deployment's URL-encoded Cloud Resource Name (CRN), and `<IAM_TOKEN>` with your Identity and Access Management (IAM) access token.

## Requirements for upgrading
{: #upgrading-reqs}

### Application compatibility
{: #app-dependencies}

Test your chosen upgrade path on a representative staging deployment before production. For an IPMVU rehearsal, restore a recent backup to the source PostgreSQL version, then upgrade it to your intended target. Check application queries, background jobs, drivers, extensions, and connection pools, and record performance for comparison afterward. Review the target version's [release notes](#changelog-postgres) and [role-privilege changes](#admin_user_issues).

### Extensions and replication objects
{: #extensions-objects}

Complete the applicable preparation below before upgrading. Plan for any application dependencies on objects that you remove.

#### Extensions
{: #extensions}

##### `pg_repack`
{: #pg_repack}

Drop `pg_repack` before the upgrade. Recreate it afterward if needed; its extension and client components must match the PostgreSQL major version.

```sh
DROP EXTENSION pg_repack;
```
{: pre}

After the upgrade:

```sh
CREATE EXTENSION pg_repack;
```
{: pre}

##### `old_snapshot`
{: #old_snapshot}

`old_snapshot` is unavailable in [PostgreSQL 17](https://www.postgresql.org/docs/17/release-17.html){: external} and later. IPMVU attempts to remove it when upgrading to these versions, without deleting dependent objects. Resolve reported dependencies before retrying. For other upgrade paths, contact support to arrange removal.

##### `anon`
{: #anon}

If `anon` is installed, run these steps as the `admin` user in each affected database.

Before removing masking rules, block database access for users who must see masked data. Keep their access blocked until you restore and verify the masking rules and role settings after the upgrade.
{: important}

1. Remove all masking rules (if enabled).

    ```sh
    SELECT anon.remove_masks_for_all_columns();
    ```
    {: pre}

2. Disable masking roles (the upgrade might fail if any roles are marked as masked).

    ```sh
    SECURITY LABEL FOR anon ON ROLE <role_name> IS NULL;
    ```
    {: pre}

3. Drop the `anon` extension with the cascade option.

    ```sh
    DROP EXTENSION anon CASCADE;
    ```
    {: pre}

4. After the upgrade, re-enable `anon`, restore its masking rules and role settings, and verify masking before restoring affected users' access.

##### `PostGIS`
{: #PostGIS}

Upgrade PostGIS before PostgreSQL:

```sh
SELECT postgis_extensions_upgrade();
```
{: pre}

Verify the extension version:

```sh
SELECT postgis_full_version();
```
{: pre}

#### Logical replication slots
{: #replication-slots}

IPMVU is blocked while any logical replication slots remain, including inactive slots and those used by `wal2json`. Before removing them:

1. Pause source writes until slots and consumers are restored after the upgrade, or plan to resynchronize downstream data.
2. Let consumers process pending changes, then stop them.
3. Drop each logical slot:

    ```sql
    SELECT pg_drop_replication_slot('<slot_name>');
    ```
    {: pre}

Do not remove service-managed physical replication slots. A new logical slot cannot recover unconsumed changes from a deleted slot.

`wal2json` is a logical decoding output plug-in, not an extension created with `CREATE EXTENSION`.

## In-place major version upgrade
{: #upgrading-in-place}

Complete [preparation](#upgrading-considerations), then use the UI, API, CLI, or Terraform procedure below.

You cannot cancel IPMVU after it starts or downgrade the deployment in place.
{: important}

### Availability during an upgrade
{: #upgrading-in-place-availability}

Compatibility checks run while the deployment is online. Database upgrade then requires a read-only period and temporary unavailability. Writes can resume before the task finishes, but further connection interruptions can occur during the remaining work.

Applications must handle connection failures and read-only errors, reconnect, and retry interrupted transactions only when safe. Connection pooling does not remove these requirements. Application access is not proof that the upgrade task completed.

### Backups and recovery
{: #upgrading-in-place-backups}

Create and verify a recent [on-demand backup](/docs/cloud-databases?topic=cloud-databases-dashboard-backups) before IPMVU; the service does not automatically take a pre-upgrade data backup. Recovery might require restoring that backup into a new deployment. Account for changes made since the backup.

The service queues a backup of the upgraded deployment to run separately after the upgrade task completes. Its duration does not extend the upgrade task. It might not start immediately. Check its status in **Backups and restore**.

Recovery of post-upgrade changes requires a successful backup of the upgraded deployment. Point-in-time recovery (PITR) also requires the corresponding transaction logs. PITR cannot replay transactions across the major version upgrade. Pre-upgrade backups and recovery points belong to the earlier version and remain subject to retention limits. See [PITR](/docs/databases-for-postgresql?topic=databases-for-postgresql-pitr).

If the queued backup fails, take an on-demand backup. Contact support if failures continue. A backup failure does not roll back the upgrade.

### Before you begin
{: #upgrading-considerations}

Apply these requirements to both staging and production:

- Complete the [application, extension, and replication preparation](#upgrading-reqs) and [backup preparation](#upgrading-in-place-backups).
- Confirm that all data members are healthy and replication is caught up. Let maintenance and other deployment changes finish.
- Keep at least 10% of allocated disk space free on each data member. Resolve sustained CPU, memory, or disk I/O pressure. Scale up your deployment as needed before starting IPMVU.
- Promote attached read-only replicas that you need to retain; delete only those no longer needed. IPMVU cannot run on a read-only replica deployment or with external replication consumers connected. Built-in high-availability members are upgraded with the source. See [Read-only replicas and IPMVU](/docs/databases-for-postgresql?topic=databases-for-postgresql-read-only-replicas#read-only-replicas-ipu).
- Have the owning application or transaction manager resolve prepared transactions. Do not commit or roll them back solely to clear a precheck.
- Choose a target from your deployment's capabilities. Availability for new deployments does not guarantee an IPMVU transition:

    ```sh
    ibmcloud cdb deployment-capability-show <NAME|CRN> versions
    ```
    {: pre}

Service prechecks cover deployment health, resources, replication, database compatibility, and transaction-log archiving before writes are interrupted. Passing prechecks does not guarantee application compatibility or prevent every later failure.

### Planning the upgrade window
{: #upgrading-in-place-window}

Use your staging rehearsal to estimate a maintenance window. Database size, object counts, workload, replication health, and additional data copying affect duration. Allow for the whole upgrade task and delays.

Expiration sets the latest time a queued upgrade can start; it does not stop a running upgrade. For example, a queued request expires at 22:30 UTC. A task started at 22:25 UTC can continue past that time.

### Upgrading in the UI
{: #upgrading-in-place-ui}
{: ui}

1. On the prepared deployment's **Overview** page, click **Upgrade major version**.
2. Select an available target and a start expiration, then submit the upgrade.
3. Monitor the task until it succeeds, then complete [After the upgrade](#upgrading-in-place-after).

### Upgrading through the API
{: #upgrading-in-place-api}
{: api}

Use an available target version from your deployment's capabilities. The following example upgrades to PostgreSQL 15 when that transition is supported:

```sh
curl -X PATCH https://api.{region}.databases.cloud.ibm.com/v5/ibm/deployments/{id}/version \
  -H 'Authorization: Bearer <IAM_TOKEN>' \
  -H 'Content-Type: application/json' \
  -d '{"version": "15"}'
```
{: pre}

Use the returned task ID with [Get task information](/apidocs/cloud-databases-api/cloud-databases-api-v5#gettask) to monitor completion. An accepted request is not a completed upgrade. For parameters, including start expiration, see the [API reference](/apidocs/cloud-databases-api/cloud-databases-api-v5#setdatabaseinplaceversionupgrade).

### Upgrading through the CLI
{: #upgrading-in-place-cli}
{: cli}

Use version 0.20.0 or later of the {{site.data.keyword.databases-for}} CLI plug-in:

```sh
ibmcloud cdb deployment-version-upgrade <NAME|CRN> <TARGET_VERSION>
```
{: pre}

Set start expiration with `--expire-in` or `--expire-at`, between 5 minutes and 24 hours from the request. Run `ibmcloud cdb deployment-version-upgrade --help` for parameters.

Monitor the task with [`deployment-tasks-list`](/docs/cli?topic=cli-cdb-reference#deployment-tasks-list):

```sh
ibmcloud cdb deployment-tasks-list <NAME|CRN>
```
{: pre}

### Upgrading through Terraform
{: #upgrading-in-place-terraform}
{: terraform}

Use {{site.data.keyword.cloud}} Terraform provider version 1.79.2 or later. Set `version` to an available target, review the plan, and apply it.

Ensure that the resource's update timeout is long enough for the operation to complete. The provider also derives the start expiration time from the configured timeout value, up to a maximum of 24 hours. For more information, see [database resource reference](https://registry.terraform.io/providers/IBM-Cloud/ibm/latest/docs/resources/database){: external}.

### After the upgrade
{: #upgrading-in-place-after}

After the upgrade task completes successfully:

1. Confirm the expected PostgreSQL version.
2. Restore extensions and masking as described in [preparation](#upgrading-reqs). Recreate required logical replication slots and read-only replicas, reconnect consumers, and complete any required data resynchronization before resuming dependent applications.
3. Review optimizer statistics after the upgrade. PostgreSQL 14 through 17 do not transfer optimizer statistics during a major version upgrade, whereas PostgreSQL 18 transfers most optimizer statistics. Follow the [PostgreSQL 17](https://www.postgresql.org/docs/17/pgupgrade.html){: external} or [PostgreSQL 18](https://www.postgresql.org/docs/18/pgupgrade.html){: external} post-upgrade instructions for your target.
4. Compare application behavior and performance with your staging results.
5. Verify the [post-upgrade backup](#upgrading-in-place-backups).

### Troubleshooting
{: #upgrading-in-place-troubleshooting}

For a precheck failure, use the reported error and [monitoring metrics](/docs/databases-for-postgresql?topic=databases-for-postgresql-monitoring) to identify the unmet [prerequisite](#upgrading-considerations), resolve it, and retry. Do not delete database objects or transaction logs just to bypass a check.

If you cannot resolve a precheck error, the upgrade fails, or the deployment remains unavailable, [contact support](https://cloud.ibm.com/unifiedsupport/supportcenter) before making further deployment changes. Include the task ID, target version, and relevant task, database, or application errors.

## Upgrading from a read-only replica
{: #upgrading-replica}

[Create a read-only replica](/docs/databases-for-postgresql?topic=databases-for-postgresql-read-only-replicas) from the source deployment and wait for the replica to synchronize. Then, promote and upgrade the replica by using the [`/remotes/promotion` endpoint](/apidocs/cloud-databases-api/cloud-databases-api-v5#promotereadonlyreplica), specifying a supported target version:

```sh
curl -X POST \
  https://api.{region}.databases.cloud.ibm.com/v5/ibm/deployments/{id}/remotes/promotion \
  -H 'Authorization: Bearer <IAM_TOKEN>' \
  -H 'Content-Type: application/json' \
  -d '{
    "promotion": {
        "version": "14",
        "skip_initial_backup": false
    }
}'
```
{: pre}

Set `skip_initial_backup` to `true` only if you want to skip the initial backup that is created during promotion. Although this setting can reduce the time required to complete the promotion task, recovery from the promoted deployment requires a later successful scheduled or on-demand backup.

### Dry running the promotion and upgrade
{: #promotion-dry-run}

A dry run validates the promotion and upgrade process without performing the actual conversion. Review the results through [log integration](/docs/cloud-databases?topic=cloud-databases-logging). A successful check does not guarantee that the actual upgrade will succeed.

Specify the target `version`, `dry_run: true` and `skip_initial_backup: false`:

```sh
curl -X POST \
  https://api.{region}.databases.cloud.ibm.com/v5/ibm/deployments/{id}/remotes/promotion \
  -H 'Authorization: Bearer <IAM_TOKEN>' \
  -H 'Content-Type: application/json' \
  -d '{
    "promotion": {
        "version": "14",
        "skip_initial_backup": false,
        "dry_run": true
    }
}'
```
{: pre}

## Back up and restore upgrade
{: #backup-restore}

[Restore a backup](/docs/cloud-databases?topic=cloud-databases-dashboard-backups&interface=ui#restore-backup-ui) to a deployment that uses a supported target version. By default, the restored deployment inherits the source deployment's disk and memory allocations from the time that the backup was created.

### Upgrading in the UI
{: #upgrading-ui}
{: ui}

From **Backups** in the deployment dashboard, click **Restore** for the backup that you want to use. Select a supported target version and configure the options for the new deployment. Then, click **Create**.

### Upgrading through the CLI
{: #upgrading-cli}
{: cli}

Create the deployment with the target version and backup ID in the `-p` JSON argument:

```sh
ibmcloud resource service-instance-create example-upgrade databases-for-postgresql standard us-south \
  -p '{
  "backup_id": "crn:v1:bluemix:public:databases-for-postgresql:us-south:a/54e8ffe85dcedf470db5b5ee6ac4a8d8:1b8f53db-fc2d-4e24-8470-f82b15c71717:backup:06392e97-df90-46d8-98e8-cb67e9e0a8e6",
  "version": "14"
}' \
  --service-endpoints "public"
```
{: pre}

### Upgrading through the API
{: #upgrading-api}
{: api}

Use the [Resource controller API](/docs/databases-for-postgresql?topic=databases-for-postgresql-provisioning&interface=api#provision-controller-api) to restore a backup to a deploymentet version. Specify the deployment name, location, resource group, plan, backup ID, and target version:

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

**Upgrade before the end-of-life date to avoid the following risks:**

- No SLAs are provided for this type of forced upgrade.
- You might experience some data loss.
- Your application might experience prolonged downtime.
- Your application might stop working if it is incompatible with the new version.
- You cannot control the timing of when this upgrade will happen for your deployment.
- There is no rollback process for this forced upgrade.

For the end-of-life dates, see the [version policy page](https://cloud.ibm.com/docs/cloud-databases?topic=cloud-databases-versioning-policy){: external}.

## Role privilege issues during version upgrades
{: #admin_user_issues}

PostgreSQL 16 and later require `ADMIN OPTION` on a role to grant or revoke its membership. Before you upgrade, review the role grants that are required for continued role management. For more information, see the [PostgreSQL 16 release notes](https://www.postgresql.org/docs/16/release-16.html){: external}, [role attributes](https://www.postgresql.org/docs/16/role-attributes.html){: external} and [role grants](https://www.postgresql.org/docs/16/sql-grant.html){: external}.

An upgraded deployment might report:

```text
ERROR: only roles with the ADMIN OPTION on role "some_role" may grant this role
```
{: pre}

For deployments upgraded from PostgreSQL 14 or 15 to PostgreSQL 16 or later, the `admin` user can run the following helper to grant the affected roles to `admin` with `ADMIN OPTION`. The helper can be run safely more than once:

```sql
SELECT grant_admin_option_to_roles('role1', 'role2', 'role3');
```
{: pre}

## Changelog for major PostgreSQL versions
{: #changelog-postgres}

- [PostgreSQL 14](https://www.postgresql.org/docs/14/release-14.html){: external}
- [PostgreSQL 15](https://www.postgresql.org/docs/release/15.0/){: external}
- [PostgreSQL 16](https://www.postgresql.org/docs/release/16.0/){: external}
- [PostgreSQL 17](https://www.postgresql.org/docs/release/17.0/){: external}
- [PostgreSQL 18](https://www.postgresql.org/docs/release/18.0/){: external}
