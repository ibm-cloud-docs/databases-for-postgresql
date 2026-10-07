---

copyright:
  years: 2019, 2026
lastupdated: "2026-10-07"

keywords: postgresql, databases, point in time recovery, backups, restore, pitr

subcollection: databases-for-postgresql

---

{{site.data.keyword.attribute-definition-list}}

# Point-in-time recovery
{: #pitr}

{{site.data.keyword.databases-for-postgresql_full}} supports point-in-time recovery (PITR) to an available recovery point within the last 7 days. PITR restores a backup into a new deployment and replays retained transaction logs to the requested time.

Point-in-time recovery (PITR) cannot replay transactions across an in-place major version upgrade (IPMVU). After an upgrade, you can restore to a point in time before the upgrade task starts, or after the first backup of the upgraded deployment completes. Restores to points between the upgrade and the completion of the next backup are not supported. Even after the backup completes, restores to some points might be rejected by the service, while restores to other points might be accepted but fail during the restore process. 

After a successful upgrade, the service automatically queues a backup. From the time writes resume until that backup completes, restores to the latest available recovery point also fail.

After a failed upgrade, check the PostgreSQL version: `SHOW server_version;`. If the command reports the source version and the deployment accepts writes, take an on-demand backup. Otherwise, contact support.
For more information, see [Backups and recovery for IPMVU](/docs/databases-for-postgresql?topic=databases-for-postgresql-upgrading#upgrading-in-place-backups).
{: important}

The _Backups and restore_ tab of your deployment's UI keeps all your PITR information under _Point-in-time recovery_.

If the requested restore time is later than the last available transaction, the restore operation fails with the message `recovery ended before configured recovery target was reached`. If your restore fails for this reason, select **Restore to the latest available point**. Alternatively, choose an earlier date and time for **Restore to a specific point in the last 7 days**. During an IPMVU and until the first backup of the upgraded deployment completes, a restore to the latest available point can fail.
{: note}

Included information is the earliest time for a PITR. To discover the earliest recovery point through the CLI, use the [`cdb postgresql earliest-pitr-timestamp`](/docs/cli?topic=cli-cdb-reference#postgresql-earliest-pitr-timestamp) command.

```sh
ibmcloud cdb postgresql earliest-pitr-timestamp <INSTANCE_NAME_OR_CRN>
```
{: pre}

To discover the earliest recovery point through the API, use the [`/deployments/{id}/point_in_time_recovery_data`](/apidocs/cloud-databases-api/cloud-databases-api-v5#getpitrdata) endpoint to find the earliest PITR time.

```sh
{
    "point_in_time_recovery_data": {
        "earliest_point_in_time_recovery_time": "2019-09-09T23:16:00Z"
    }
}
```
{: .codeblock}

## Recovery
{: #recovery}

Backups are restored to a new deployment. After the new deployment finishes provisioning, your data in the backup file is restored into the new deployment. Backups are also restorable across accounts, but only by using the API and only if the user that is running the restore has access to both the source and destination accounts. 

By default the new deployment is auto-sized to the same disk and memory allocation as the source deployment at the time of the backup that you are restoring from. Especially in the case of PITR that might not be the current size of your deployment. If you need to adjust the resources that are allocated to the new deployment, use the optional fields in the UI, CLI, or API to resize the new deployment. Be sure to allocate enough for your data and workload, if the deployment is not given enough resources the restore fails.

While storage and memory are restored to the same as the source deployment, specific instance configurations are not automatically set for the new instance. In this case, rerunning the configuration after a restore might be needed. Note any instance modifications before running the restore (parameters, such as `shared_buffers`, `max_connections`, `deadlock_timeout`, `archive_timeout`, and others) to ensure accurate setting for the instance after the restore is complete.

It is important that you do not delete the source deployment while the backup is restoring. You must wait until the new deployment is provisioned and the backup is restored before deleting the old deployment. Deleting a deployment also deletes its backups so not only does the restore fail, you might not be able to recover the backup either.
{: .tip}

### Recovery in the UI
{: #pitr-ui}
{: ui}

To initiate a PITR, enter the time that you want to restore back to in Coordinated Universal Time. If you want to restore to the most recent available time, select that option. Clicking the **Restore** button brings up the new provisioning UI in a tab with the options for your recovery. Enter the service details, allocate resources, and set the database version, encryption and endpoint for your new deployment. Click **Point in time recovery** to start the process.

If you use Key Protect and have a key, you must use the CLI to recover, and a command is provided for your convenience.

### Recovery in the CLI
{: #pitr-cli}
{: cli}

The Resource controller supports provisioning of database deployments, and provisioning and restoring are the responsibility of the Resource controller CLI. Use the [`resource service-instance-create`](/docs/cli?topic=cli-ibmcloud_commands_resource#ibmcloud_resource_service_instance_create) command.

For PITR, use the `point_in_time_recovery_time` and `point_in_time_recovery_deployment_id` parameters. The `point_in_time_recovery_deployment_id` is the source deployment's ID and `point_in_time_recovery_time` is the timestamp in Coordinated Universal Time you want to restore to. To restore to the latest available point-in-time, use `"point_in_time_recovery_time":" "`.

```sh
ibmcloud resource service-instance-create <databases-for-postgresql> <INSTANCE_NAME> <REGION> -p '{"point_in_time_recovery_deployment_id":"DEPLOYMENT_ID", "point_in_time_recovery_time":"TIMESTAMP", "version":" "}'
```
{: pre}

A pre-formatted command for a specific backup or PITR is available in detailed view of the backup.
{: .tip}

When restoring through the CLI, optional parameters are available. Use them to customize resources or use a Key Protect key for BYOK encryption on the new deployment.

```sh
ibmcloud resource service-instance-create <databases-for-postgresql> <INSTANCE_NAME> standard <REGION> <--service-endpoints SERVICE_ENDPOINTS_TYPE> -p
'{"point_in_time_recovery_deployment_id":"DEPLOYMENT_ID", "point_in_time_recovery_time":"TIMESTAMP","key_protect_key":"KEY_PROTECT_KEY_CRN", "members_disk_allocation_mb":"DESIRED_DISK_IN_MB", "members_memory_allocation_mb":"DESIRED_MEMORY_IN_MB", "members_cpu_allocation_count":"NUMBER_OF_CORES", "version":" "}'
```
{: pre}

### Recovery in the API
{: #pitr-api}
{: api}

The Resource Controller supports provisioning of database deployments, and provisioning and restoring are the responsibility of the Resource Controller API. You need to complete [the necessary steps to use the resource controller API](/apidocs/resource-controller/resource-controller) before you can use it to restore from a backup. 

Once you have all the information, the create request is a `POST` to the [`/resource_instances`](https://{DomainName}/apidocs/resource-controller#create-provision-a-new-resource-instance) endpoint.

```sh
curl -X POST \
  https://resource-controller.cloud.ibm.com/v2/resource_instances \
  -H 'Authorization: Bearer <>' \
  -H 'Content-Type: application/json' \
    -d '{
    "name": "<INSTANCE_NAME>",
    "target": "<REGION>",
    "resource_group": "<RESOURCE_GROUP>",
    "resource_plan_id": "<SERVICE_ID>",
    "parameters": {
      "point_in_time_recovery_time": "<TIMESTAMP>",
      "point_in_time_recovery_deployment_id": "<DEPLOYMENT_ID>"
    }
  }'
```
{: pre}

The parameters `name`, `target`, `resource_group`, and `resource_plan_id` are all required. The `target` is the region where you want the new deployment to be located, which can be a different region from the source deployment. Cross region restores are supported, except for restoring a `eu-de` backup to another region.

For PITR, use the `point_in_time_recovery_time` and `point_in_time_recovery_deployment_id` parameters. The `point_in_time_recovery_deployment_id` is the source deployment's ID and `point_in_time_recovery_time` is the timestamp in Coordinated Universal Time you want to restore to. To restore to the latest available point-in-time use `"point_in_time_recovery_time":" "`.

Pass these parameters in the `parameters` object of the request body. To customize resource allocations or use a Key Protect key, add the applicable parameters to the same object: `key_protect_key`, `members_disk_allocation_mb`, `members_memory_allocation_mb`, and `members_cpu_allocation_count`.

## Verifying PITR
{: #pitr-verify}

To verify the correct recovery time, check the database logs. Checking the database logs requires the [Logging integration](/docs/cloud-databases?topic=cloud-databases-logging) to be set up on your deployment.

When you perform a recovery, your data is restored from the most recent incremental backup. Any outstanding transactions from the WAL log are used to restore your database up to the time you recovered to. After the recovery is finished, and the transactions are run, the logs display a message like the following:

```text
LOG:  last completed transaction was at log time 2019-09-03 19:40:48.997696+00
```
{: codeblock}

Recovery does not show up in the logs if your deployment has a recent full backup and no activity after that backup needs to be replayed. In this case, the recovery is usually still successful, but the logs have no entry that shows the exact time that the database was restored to.
