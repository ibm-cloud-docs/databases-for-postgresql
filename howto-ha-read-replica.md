---

copyright:
  years: 2023
lastupdated: "2023-06-12"

keywords: postgresql, databases, ha read-only replica, high availability read-only replica, resync, promote, cross-region replication, postgres replica, postgresql replica, leader deployment, read replica, data member, replication status

subcollection: databases-for-postgresql

---

{{site.data.keyword.attribute-definition-list}}

# The {{site.data.keyword.databases-for-postgresql}} high-availability read-only replica
{: #the-ha-read-only-replica}

A high-availability {{site.data.keyword.databases-for-postgresql}} read-only replica provides benefits such as improved read scalability, increased availability, reduced read latency, backup and disaster recovery capabilities, and the ability to distribute read traffic efficiently. It contributes to a more robust and responsive database infrastructure for your application.

## Provisioning a high-availability read-only replica
{: #provision-ha-read-only-replica}

High-availability {{site.data.keyword.databases-for-postgresql}} read-only replicas must initially be provisioned in the single-member (non-HA) configuration. For more information, see [Provisioning a Read-only Replica](/docs/databases-for-postgresql?topic=databases-for-postgresql-read-only-replicas&interface=ui#read-only-replicas-provision). 

Provisioning a high-availability read-only replica can be done through the [{{site.data.keyword.databases-for}} API](https://cloud.ibm.com/apidocs/cloud-databases-api/cloud-databases-api-v5#introduction). After provisioning, use the API, as in the following example, to scale up the read replica to two (or more) members.

Use a command like, 

```sh
curl -XPATCH -H 'Authorization: Bearer <>' "https://api.test-yp-01.us-south.databases.cloud.ibm.com/v5/ibm/deployments/{id}/groups/member" -d '{"members": {"allocation_count": 2}}'
```
{: pre}

The `{id}` value is your URL-encoded CRN. For more information, see [Deployment IDs and CRNs](https://cloud.ibm.com/apidocs/cloud-databases-api/cloud-databases-api-v5#deployment-ids-and-crns).
{: note}

## Provisioning a Read-only Replica
{: #read-only-replicas-provision}

Resources for PostgreSQL deployments are allocated per-deployment, and normal deployments have two members. Since a read-only replica has only one member, and provisioning currently uses values that are half of the requested values for memory and storage, provisioning can fail. The web UI cannot modify the value for storage and it automatically uses the leader deployment’s value - which is cut in half. If that is insufficient for your data to fit, you need to use the API or CLI to specify twice the storage you want to be provisioned. (The same applies to memory, although a lower amount of memory might not prevent the restore from being successful.) An update is in progress to remediate this situation.
{: important}


