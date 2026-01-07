---
copyright:
  years: 2020, 2026
lastupdated: "2026-01-07"

keywords: postgresql, databases, upgrading, major versions, postgresql new deployment, postgresql database version, postgresql major version

subcollection: databases-for-postgresql

---

{{site.data.keyword.attribute-definition-list}}

# Upgrading to a new major version
{: #upgrading}

As of December 2025, {{site.data.keyword.databases-for-postgresql}} provides three different upgrade paths:

- In-place upgrade to a new major version.
- Restoring from backup.
- Upgrading from a read-only replica.

When a major version of a database is nearing its End Of Life (EOL), it is advisable to upgrade to a current major version. 

Find the available versions of {{site.data.keyword.databases-for-postgresql}} in the [{{site.data.keyword.cloud_notm}} catalog](https://cloud.ibm.com/databases/databases-for-postgresql/create) page, from the {{site.data.keyword.databases-for}} CLI plug-in command [`ibmcloud cdb deployables-show`](/docs/databases-cli-plugin?topic=databases-cli-plugin-cdb-reference#deployables-show){: external}, or from the {{site.data.keyword.databases-for}} API [`/deployables`](/apidocs/cloud-databases-api/cloud-databases-api-v5#listdeployables){: external} endpoint.

When you upgrade to a new instance, you also need to change the connection information in your application.
{: note}

In the following example commands, the full CRN of the database instance is required for the {id}. Since the CRN contains special characters, it must be *URL-encoded* to avoid a "not_found" error.
{: note}


## Requirements for upgrading to newer PostgreSQL major version from PostgreSQL v14 
{: #upgrading-reqs}

If you have `pg_repack` installed, you need to remove it before performing the upgrade. This can be done with a command like:

```sh
DROP EXTENSION pg_repack; 
```
{: pre}

After upgrading, reinstall `pg_repack`. This can be done with the following command:

```sh
CREATE EXTENSION pg_repack;
```
{: pre}

If you are using PostGIS, you must first upgrade to PostGIS before upgrading PostgreSQL. This can be done by running the following command against a database with PostGIS installed.
{: note}

```sh
SELECT postgis_extensions_upgrade();
```
{: pre}

Use the following query to validate the PostGIS extension upgrade. 

```sh
SELECT postgis_full_version();
```
{: pre}

## In-place major version upgrades
{: #upgrading-in-place}

In-place major version upgrade allows you to upgrade your deployment to the next new [major version](/docs/databases-for-postgresql?topic=databases-for-postgresql-versioning-policy#version-definitions), eliminating the need to [restore a backup](/docs/databases-for-postgresql?topic=databases-for-postgresql-upgrading&interface=ui#backup-restore) into a new deployment. This approach maintains the same connection strings, without the need to reconfigure the deployment. However, if the new major version requires application adjustments, these must be addressed.

During the in-place major version upgrade window, your deployment will experience a brief period of downtime. This is expected, as the process follows the vendor-recommended upgrade approach. The exact duration may vary depending on the size and complexity of your deployment’s schema. If your service needs to read data from the upgraded instance during this time, you may want to [create a standby instance](docs/databases-for-postgresql?topic=databases-for-postgresql-read-only-replicas&interface=ui#read-only-replicas-provision) and update your application’s connection details to point to the standby. This ensures you have an up-to-date copy of your database prior to starting the upgrade. The standby instance can also be promoted and used as a primary instance if the in-place upgrade does not complete successfully. Additional information is available on the read-only replica page and can be seen [here](docs/databases-for-postgresql?topic=databases-for-postgresql-read-only-replicas&interface=ui#read-only-replicas-ipu).
{: important}

{{site.data.keyword.databases-for-postgresql}} gives customers flexibility in managing their own backups. The in-place major version upgrade process does not automatically create a backup before or after the task. If the upgrade is not successful, you may need to restore your deployment from your most recent backup into a new instance.

While in-place upgrades are supported, performing them without a recent backup is not advised. Having a current backup before and after initiating the upgrade provides an added layer of protection and helps ensure a smooth recovery path if needed. Hence, consider having a fresh backup before and after the in-place major version upgrade task.
{: important}

### Before you begin
{: #upgrading-considerations}

Consider the following aspects before starting the upgrade procedure.

- Check if version upgrade is available for your deployment version by checking the deployment capability information via UI, API, CLI, or Terraform.

  Example: Looking up version upgrade information with CLI.

    ```bash
    ibmcloud cdb capability-show versions postgresql
    ```
- Your deployment must be in a healthy state before upgrading.
- Your deployment must have at least 10% of your disk space available.
- Your deployment must not be under heavy I/O pressure (max: 90% allowed).
- You can only upgrade from PostgreSQL 14 to the PostgreSQL 15 at this stage.
- Each major version contains some features that may not be backward-compatible with previous versions. Check the [release notes](https://www.postgresql.org/docs/release/) from the database vendor to see any changes that may affect your applications.
- Downgrading a deployment to a previous version is not supported.
- In-place major version upgrade cannot be cancelled once it started.
- If you do not have a fresh backup, consider to take one before upgrading.

Also note that after the upgrade completes, your database will be running a new PostgreSQL major version. Because PostgreSQL stores data in version-specific formats, the new PostgreSQL data directory cannot be used with earlier versions. This means that backups taken before the upgrade cannot be used to restore into the new PostgreSQL version. To maintain full restore and PITR (Point-in-Time Recovery) capabilities going forward, take a new backup after the upgrade. This new backup becomes the baseline for all future recovery operations in the PostgreSQL timeline.
{: important}

### Upgrading in the UI
{: #upgrading-in-place-ui}
{: ui}

1. Create a new {{site.data.keyword.databases-for-postgresql}} to test the upgrade process. <br> Create the deployment by [restoring a backup](/docs/cloud-databases?topic=cloud-databases-dashboard-backups&interface=ui#restore-backup) from your existing deployment with the same version.
2. Point your staging application to the test deployment. <br> Update your staging application to point to the test deployment. Confirm that your test application can connect successfully to the staging deployment and that the application operates as expected. Perform any required performance and operational testing of the staging environment.
3. Upgrade the major version of your test deployment by clicking on the **Upgrade major version** button on the *Overview* page. <br> Note how long the upgrade takes to complete, so that you can use the upgrade expiry setting to contain upgrades within your maintenance window.
4. Confirm that your staging application works with the new database version. <br> If your application works, this step confirms that it should be safe to upgrade your production database.
5. Upgrade your production database deployment to the new version. <br> Once you confirmed that your application works correctly by using the new version of the database, you can return to the management console and start the process of upgrading your production deployment. In the **Deployment details** section of the *Overview* page, click the **Upgrade major version** button and follow the steps.

   Once the in-place upgrade process starts, it cannot be stopped or rolled back. So, in the unlikely event of an error, your database deployment could become unrecoverable. Therefore, create a backup that you can then use to restore to a new deployment.

The `expiration for starting upgrade` allows you to configure a 'timeout' period that the upgrade job must start within before it is automatically cancelled. In addition, test the upgrade in staging upfront to ensure that the upgrade completes within your desired time window. If, for example, you want to complete the upgrade within 1 hour, and you tested the upgrade and know that it takes 30 minutes, then your upgrade job must start within 30 minutes of you confirming that you want to upgrade. Therefore, set the expiration to 30 minutes, so that if it doesn't start within that time, it won't overrun your window.

### Upgrading through the API
{: #upgrading-in-place-api}
{: api}

Use the following command to upgrade in-place:

```sh
curl -X PATCH https://api.{region}.databases.cloud.ibm.com/v5/ibm/deployments/{id}/version -H 'Authorization: Bearer <>' -H 'Content-Type: application/json' -d '{"version": "15"}'
```
{: pre}

The `expiration for starting upgrade` allows you to configure a 'timeout' period that the upgrade job must start within before it is automatically cancelled. In addition, test the upgrade in staging upfront to ensure that the upgrade completes within your desired time window. If, for example, you want to complete the upgrade within 1 hour, and you tested the upgrade and know that it takes 30 minutes, then your upgrade job must start within 30 minutes of you confirming that you want to upgrade. Therefore, set the expiration to a timestamp of 30 minutes from now, so that if it doesn't start within that time, it won't overrun your window. The expiration must be between 5 minutes (default) and 24 hours from now. For more information, see the [Cloud Databases API](https://cloud.ibm.com/apidocs/cloud-databases-api/cloud-databases-api-v5#setdatabaseinplaceversionupgrade).

### Upgrading through the CLI
{: #upgrading-in-place-cli}
{: cli}

Available in CDB plugin version >= 0.20.0. 
{: .note}

To view the list of allowed upgrade and restore transitions for the deployment:

```sh
ibmcloud cdb deployment-capability-show <NAME|CRN> versions
```

To upgrade the command with required parameters:

```sh
ibmcloud cdb deployment-version-upgrade <NAME|CRN> <TARGET_VERSION>
```

To view full details of command parameters:

```sh
ibmcloud cdb deployment-version-upgrade --help
```

The `expiration for starting upgrade` allows you to configure a 'timeout' period that the upgrade job must start within before it is automatically cancelled. In addition, test the upgrade in staging upfront to ensure that the upgrade completes within your desired time window. If, for example, you want to complete the upgrade within 1 hour, and you tested the upgrade and know that it takes 30 minutes, then your upgrade job must start within 30 minutes of you confirming that you want to upgrade. Therefore, set the expiration to 30 minutes, so that if it doesn't start within that time, it won't overrun your window. The  expiration must be between 5 mins (default) and 24 hours from now. There are two ways to set expiration using CLI `--expire-in` or `--expire-at`. For more information, refer to the command help.

### Upgrading through Terraform
{: #upgrading-in-place-terraform}
{: terraform}

Available in Terraform provider version >= 1.79.2 .
{: .note}

To upgrade, just add or change the `version` value in your configuration.

Skipping a backup before a version upgrade is dangerous and may result in data loss if the upgrade fails at any stage — there will be no immediate backup to restore from. Hence, consider to have a fresh backup before initiating an in-place major version upgrade task.
{: .attention}

Upgrading may require more time than the default timeout. A longer timeout value can be set with using the timeouts attribute.
{: .note}

Terraform has timeouts instead of expiration timestamps. Therefore, increase your timeout, as your timeout update value is used as the expiration. For example, if you set a timeout of 20 minutes, the expiration will be set to 20 minutes and if the upgrade does not start in that time frame, it expires and the upgrade will not start. Note that the maximum expiration is 24 hours — so even if you set a timeout of 36 hours, the upgrade will expire if it hasn't started within the first 24 hours.

If an upgrade is in progress, note that some tasks may be queued and will not proceed until the version upgrade completes.

### Troubleshooting
{: #upgrading-in-place-troubleshooting}

If your applications show unexpected issues after a successful in-place major version upgrade and you need to return to the previous PostgreSQL version, contact our support team for guidance. Avoid initiating a PITR or restoring a backup on your own, as this may complicate recovery.
{: .attention}

An in-place major upgrade will not proceed until all prechecks have passed. These safeguards exist to protect your deployment, since the upgrade operates directly on the source instance. If the upgrade is blocked, review the following areas:

-  Cluster state: Ensure that the Patroni cluster is healthy and that a clear leader/replica state is present. Upgrades cannot proceed if Patroni reports instability or failover conditions.
- Disk space: Verify that sufficient free space is available. The process uses `pg_upgrade` link mode, which requires adequate headroom. If disk usage exceeds the configured limit (default: 90%), free space before retrying.
- Disk I/O load: Check current I/O utilization and IOPS. Upgrades are paused when the system is under heavy load to prevent performance degradation or upgrade failure.
- Schema size and object count: As mentioned, schema size directly affects the duration of an in-place major version upgrade. Ensure that no individual schema exceeds the maximum size (default: 100 GB) and that the total number of index and sequence objects remains below the limit (default: 50,000). Large schemas or unusually high object counts may require cleanup or optimization before proceeding with the upgrade.
`pg_upgrade` performs rapid upgrades by creating new system tables and simply reusing the old user data files. The time required to create these system tables scales with the number of database objects.
The resource consumption can be evaluated by using the [monitoring integration](/docs/databases-for-postgresql?topic=databases-for-postgresql-monitoring&interface=ui). If not all database components are available to be upgraded, the upgrade task fails. This can happen due to maintenance. Tasks that failed due to failed healthchecks can be retried later. If the task continuously fails, open a support ticket with [IBM Cloud support](https://cloud.ibm.com/login?redirect=%2Funifiedsupport%2Fsupportcenter).
If certain checks are not relevant to your environment and the upgrade is still blocked, create a support ticket for further assistance.

## Handling the `anon` extension before upgrade
{: #upgrading-handling}

If the `anon` extension is installed, follow the steps below and execute the commands as the admin user before performing the upgrade.

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

4. If the `anon` extension is installed in multiple databases within an instance, follow the outlined steps for each database.

5. Post-upgrade steps:

   After the upgrade is complete, re-enable the `anon` extension and reapply masking rules as required.

   It is strongly recommended to validate the data both before and after dropping the extension to ensure masking consistency prior to performing the upgrade.
   {: note}

## Upgrading from a read-only replica
{: #upgrading-replica}

Upgrade by [configuring a read-only replica](/docs/databases-for-postgresql?topic=databases-for-postgresql-read-only-replicas). Provision a read-only replica with the same database version as your deployment and wait while it replicates all of your data. When your deployment and its replica are synced, promote and upgrade the read-only replica to a full, stand-alone deployment running the new version of the database. To perform the upgrade and promotion step, use a POST request to the [`/deployments/{id}/remotes/promotion`](/apidocs/cloud-databases-api/cloud-databases-api-v5#promotereadonlyreplica) endpoint with the version that you want to upgrade to in the body of the request. 

This request looks like:

```sh
curl -X POST \
  https://api.{region}.databases.cloud.ibm.com/v5/ibm/deployments/{id}/remotes/promotion \
  -H 'Authorization: Bearer <>'  \
 -H 'Content-Type: application/json' \
 -d '{
    "promotion": {
        "version": "14",
        "skip_initial_backup": false
    }
}' \
```
{: pre}

`skip_initial_backup` is optional. If set to `true`, the new deployment does not take an initial backup when the promotion completes. Your new deployment is available in a shorter amount of time, at the expense of not being backed up until the next automatic backup is run, or you take an on-demand backup.

### Dry running the promotion and upgrade
{: #promotion-dry-run}

To evaluate the effects of major version upgrades, trigger a dry run. A dry run simulates the promotion and upgrade, with the results printed to the database logs. Access and view your database logs through the [Log analysis integration](/docs/cloud-databases?topic=cloud-databases-logging). This ensures the version that you are currently running with its extensions can be successfully upgraded to your intended version.

The dry run must be run with `skip_initial_backup` set to `false`, and `version` defined.

The command looks like:

```sh
curl -X POST \
  https://api.{region}.databases.cloud.ibm.com/v5/ibm/deployments/{id}/remotes/promotion \
  -H 'Authorization: Bearer <>'  \
 -H 'Content-Type: application/json' \
 -d '{
    "promotion": {
        "version": "14",
        "skip_initial_backup": false,
        "dry_run": true
    }
}' \
```
{: pre}

## Back up and restore upgrade
{: #backup-restore}

You can upgrade your database version by [restoring a backup](/docs/cloud-databases?topic=cloud-databases-dashboard-backups&interface=ui#restore-backup-ui) of your data into a new deployment that is running the new database version.

### Upgrading in the UI
{: #upgrading-ui}
{: ui}

Upgrade to a new version when [restoring a backup](/docs/cloud-databases?topic=cloud-databases-dashboard-backups&interface=ui#restore-backup-ui) from the _Backups_ menu of your _Deployment dashboard_. Click **Restore** on a backup to the provisioning page on a new tab, where you can change some options for the new deployment. One of the options is the database version, which is auto-populated with the versions available for you to upgrade to. Select a version and click **Create** to start the provision and restore process.

### Upgrading through the CLI
{: #upgrading-cli}
{: cli}

To upgrade and restore from a backup through the {{site.data.keyword.cloud_notm}} CLI, use the provisioning command from the resource controller.

```sh
ibmcloud resource service-instance-create <DEPLOYMENT_NAME_OR_CRN> <SERVICE_ID> <SERVICE_PLAN_ID> <REGION>
```
{: pre}

The parameters `service-name`, `service-id`, `service-plan-id`, and `region` are all required. You also supply the `-p` with the version and backup ID parameters in a JSON object. The new deployment is automatically sized with the same disk and memory as the source deployment at the time of the backup.

This command looks like:

```sh
ibmcloud resource service-instance-create example-upgrade databases-for-postgresql standard us-south \
-p \ '{
  "backup_id": "crn:v1:bluemix:public:databases-for-postgresql:us-south:a/54e8ffe85dcedf470db5b5ee6ac4a8d8:1b8f53db-fc2d-4e24-8470-f82b15c71717:backup:06392e97-df90-46d8-98e8-cb67e9e0a8e6",
  "version":14
}'
```
{: pre}

### Upgrading through the API
{: #upgrading-api}
{: api}

Complete the necessary steps to use the [Resource controller API](/docs/databases-for-postgresql?topic=databases-for-postgresql-provisioning&interface=api#provision-controller-api) before you use it to upgrade from a backup. Then, send the API a `POST` request. The parameters `name`, `target`, `resource_group`, and `resource_plan_id` are all required. You also supply the version and backup ID. The new deployment has the same memory and disk allocation as the source deployment at the time of the backup. 

This command looks like:

```sh
curl -X POST \
  https://resource-controller.cloud.ibm.com/v2/resource_instances \
  -H 'Authorization: Bearer <>' \
  -H 'Content-Type: application/json' \
    -d '{
    "name": "my-instance",
    "target": "bluemix-us-south",
    "resource_group": "5g9f447903254bb58972a2f3f5a4c711",
    "resource_plan_id": "databases-for-postgresql-standard",
    "backup_id": "crn:v1:bluemix:public:databases-for-postgresql:us-south:a/54e8ffe85dcedf470db5b5ee6ac4a8d8:1b8f53db-fc2d-4e24-8470-f82b15c71717:backup:06392e97-df90-46d8-98e8-cb67e9e0a8e6",
    "version":14
  }'
```
{: pre}

## Forced upgrade
{: #forced_upgrade}

After the end-of-life date, all active {{site.data.keyword.databases-for-postgresql}} deployments on the deprecated version will be forcibly upgraded to the next supported version. For example, PostgreSQL Version 13 (deprecated) upgrades to Version 14.
{: .note}

**Upgrade before the end-of-life date to avoid the following risks:**

- No SLAs are provided for this type of forced upgrade.
- You may experience some data loss.
- Your application may experience prolonged downtime.
- Your application may stop working if it is incompatible with the new version.
- You cannot control the timing of when this upgrade will happen for your deployment.
- There is no rollback process for this forced upgrade.

For the end-of-life dates, refer to the [version policy page](https://cloud.ibm.com/docs/cloud-databases?topic=cloud-databases-versioning-policy ){: external}.

## _Admin user_ issues during version upgrades
{: #admin_user_issues}

Starting with PostgreSQL 16, role privilege enforcement has become more stringent. In contrast, earlier versions allowed roles with the `CREATEROLE` attribute to manage other roles; version 16 and beyond require that a role has the `ADMIN OPTION` on another role to grant or revoke it.
{: .note}

**If you are upgrading your database from PostgreSQL v15 or earlier to PostgreSQL v16 or higher, you may encounter privilege-related errors, such as:**

```sh
ERROR: only roles with the ADMIN OPTION on role "some_role" may grant this role
DETAIL: role "admin" is not permitted to grant role "some_role"
```
{: pre}

To address this issue, we provide a built-in helper function `grant_admin_option_to_roles` that is designed to restore the expected role management behavior.

This function:

- Applies only to databases upgraded from PostgreSQL v15 and earlier versions to PostgreSQL 16 and later (if you're encountering the error described above).
- Accepts an arbitrary list of roles to apply the fix to.
- Can only be executed by the `admin user`.
- Is safe to run multiple times (idempotent).

Sample usage:

```sh
SELECT grant_admin_option_to_roles('role1', 'role2', 'role3');
```
{: pre}

This ensures that the `admin user` retains the appropriate `ADMIN OPTION` privileges to manage designated roles in upgraded instances.

## Changelog for major PostgreSQL versions
{: #changelog-postgres}

- [PostgreSQL 13](https://www.postgresql.org/docs/13/release-13.html){: external}
- [PostgreSQL 14](https://www.postgresql.org/docs/14/release-14.html){: external}
- [PostgreSQL 15](https://www.postgresql.org/docs/release/15.0/){: external}
- [PostgreSQL 16](https://www.postgresql.org/docs/release/16.0/){: external}
- [PostgreSQL 17](https://www.postgresql.org/docs/release/17.0/){: external}
