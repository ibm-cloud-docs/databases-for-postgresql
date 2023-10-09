---

copyright:
  years: 2023
lastupdated: "2023-10-09"

keywords: postgresql, databases, ha read-only replica, high availability read-only replica, resync, promote, cross-region replication, postgres replica, postgresql replica, leader deployment, read replica, data member, replication status

subcollection: databases-for-postgresql

---

{{site.data.keyword.attribute-definition-list}}

# The high-availability read-only replica
{: #the-ha-read-only-replica}

A high-availability {{site.data.keyword.databases-for-postgresql}} read-only replica provides benefits such as improved read scalability, increased availability, reduced read latency, backup and disaster recovery capabilities, and the ability to distribute read traffic efficiently. It contributes to a more robust and responsive database infrastructure for your application.

## Provision a high-availability read-only replica
{: #provision-ha-read-only-replica}

High-availability {{site.data.keyword.databases-for-postgresql}} read-only replicas must initially be provisioned in the single-member (non-HA) configuration. To provision a read-only replica, follow the procedure that is outlined in [Provisioning a Read-only Replica](/docs/databases-for-postgresql?topic=databases-for-postgresql-read-only-replicas&interface=ui#read-only-replicas-provision).

After your read-replica is provisioned, use the [{{site.data.keyword.databases-for}} API](https://cloud.ibm.com/apidocs/cloud-databases-api/cloud-databases-api-v5#introduction) to scale up the read replica to two (or more) members.

To scale up your read-replica, use the [Scaling groups endpoint](https://cloud.ibm.com/apidocs/cloud-databases-api/cloud-databases-api-v5#setdeploymentscalinggroup) of the {{site.data.keyword.databases-for}} API.

Use a command like:

```sh
curl -XPATCH -H 'Authorization: Bearer <>' "https://api.{region}.us-south.databases.cloud.ibm.com/v5/ibm/deployments/{id}/groups/member" -d '{"members": {"allocation_count": 2}}'
```
{: pre}

- The `{region}` is 
- The `{id}` value is your URL-encoded CRN. For more information, see [Deployment IDs and CRNs](https://cloud.ibm.com/apidocs/cloud-databases-api/cloud-databases-api-v5#deployment-ids-and-crns).

## Verifying the state of your read-replica
{: #verifying-ha-read-only-replica}

To verify the current state of your read-replica, use the [Scaling groups endpoint](https://cloud.ibm.com/apidocs/cloud-databases-api/cloud-databases-api-v5#listdeploymentscalinggroups) of the {{site.data.keyword.databases-for}} API.

Use a command like:

```sh
curl -XGET -H 'Authorization: Bearer <>' "https://api.us-south.databases.cloud.ibm.com/v5/ibm/deployments/{id}/groups"
```
{: pre}
