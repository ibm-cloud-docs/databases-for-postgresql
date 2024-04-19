---
copyright:
  years: 2020, 2024
lastupdated: "2024-04-19"

keyowrds: postgresql, databases, upgrading, major versions, postgresql new deployment, postgresql database version, postgresql major version

subcollection: databases-for-postgresql

---

{{site.data.keyword.attribute-definition-list}}

# Upgrading to a new Major Version
{: #upgrading}

Once a major version of a database is at its End Of Life (EOL), it is a good idea to upgrade to a current major version. 

Find the available versions of {{site.data.keyword.databases-for-postgresql}} in the [{{site.data.keyword.cloud_notm}} catalog](https://cloud.ibm.com/catalog/databases-for-postgresql) page, from the {{site.data.keyword.databases-for}} CLI plug-in command [`ibmcloud cdb deployables-show`](/docs/databases-cli-plugin?topic=databases-cli-plugin-cdb-reference#deployables-show){: external}, or from the {{site.data.keyword.databases-for}} API [`/deployables`](https://cloud.ibm.com/apidocs/cloud-databases-api#get-all-deployable-databases){: external} endpoint.

## Requirements for upgrading to PostgreSQL (v 13, 14, 15) from PostgreSQL (v10, 11, 12)
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

If you are using PostGIS, you must upgrade to PostGIS 3.3 before upgrading. This can be done by running the following against a database with PostGIS installed.
{: note}

```sh
SELECT * FROM update_to_postgis_33();
```
{: pre}


## Upgrading from a Read-only Replica
{: #upgrading-replica}

Upgrade by [configuring a read-only replica](/docs/databases-for-postgresql?topic=databases-for-postgresql-read-only-replicas). Provision a read-only replica with the same database version as your deployment and wait while it replicates all of your data. Once your deployment and its replica are synced, promote and upgrade the read-only replica to a full, stand-alone deployment running the new version of the database. To perform the upgrade and promotion step, use a POST request to the [`/deployments/{id}/remotes/promotion`](https://cloud.ibm.com/apidocs/cloud-databases-api#promote-read-only-replica-to-a-full-deployment) endpoint with the version that you want to upgrade to in the body of the request. 

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

## Back up and Restore Upgrade
{: #backup-restore}

You can upgrade your database version by [restoring a backup](/docs/databases-for-postgresql?topic=databases-for-postgresql-dashboard-backups&interface=ui#restore-backup) of your data into a new deployment that is running the new database version.

### Upgrading in the UI
{: #upgrading-ui}
{: ui}

Upgrade to a new version when [restoring a backup](/docs/databases-for-postgresql?topic=databases-for-postgresql-dashboard-backups&interface=ui#restore-backup) from the _Backups_ menu of your _Deployment dashboard_. Click **Restore** on a backup to bring up a dialog box where you can change some options for the new deployment. One of the options is the database version, which is auto-populated with the versions available for you to upgrade to. Select a version and click **Restore** to start the provision and restore process.

### Upgrading through the CLI
{: #upgrading-cli}
{: cli}

To upgrade and restore from a backup through the {{site.data.keyword.cloud_notm}} CLI, use the provisioning command from the resource controller.
```sh
ibmcloud resource service-instance-create <service-name> <service-id> <service-plan-id> <region>
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

Complete the necessary steps to use the [Resource Controller API](/docs/databases-for-postgresql?topic=databases-for-postgresql-provisioning&interface=api#provision-controller-api) before you use it to upgrade from a backup. Then, send the API a `POST` request. The parameters `name`, `target`, `resource_group`, and `resource_plan_id` are all required. You also supply the version and backup ID. The new deployment has the same memory and disk allocation as the source deployment at the time of the backup. 

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

### Dry running the promotion and upgrade
{: #promotion-dry-run}

To evaluate the effects of major version upgrades, trigger a dry run. A dry run simulates the promotion and upgrade, with the results printed to the database logs. Access and view your database logs through the [Log Analysis Integration](/docs/databases-for-postgresql?topic=databases-for-postgresql-logging). This ensures the version that you are currently running with its extensions can be successfully upgraded to your intended version.

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

## Changelog for Major PostgreSQL Versions
{: #changelog-postgres}

- [PostgreSQL 10](https://www.postgresql.org/docs/10/release-10.html){: external}
- [PostgreSQL 11](https://www.postgresql.org/docs/11/release-11.html){: external}
- [PostgreSQL 12](https://www.postgresql.org/docs/current/release-12.html){: external}
- [PostgreSQL 13](https://www.postgresql.org/docs/13/release-13.html){: external}
- [PostgreSQL 14](https://www.postgresql.org/docs/14/release-14.html){: external}
- [PostgreSQL 15](https://www.postgresql.org/docs/release/15.0/){: external}
