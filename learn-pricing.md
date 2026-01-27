---
copyright:
  years: 2017, 2026
lastupdated: "2026-01-27"

keywords: postgresql pricing

subcollection: databases-for-postgresql

---

{{site.data.keyword.attribute-definition-list}}

# Pricing
{: #pricing}

A {{site.data.keyword.databases-for-postgresql}} Standard plan deploys as one highly available PostgreSQL cluster with two data members, with your data replicated on both members. It's priced based on the total amount of disk storage, RAM and backup storage that is allocated to deployments, prorated hourly. {{site.data.keyword.databases-for-postgresql}} deployments have a minimum of 5 GB of disk and 4 GB of RAM per data member.

## Using the pricing calculator
{: #pricing-calc}

For pricing estimation, use the **Add to estimate** button on the [{{site.data.keyword.databases-for-postgresql}} catalog page](https://cloud.ibm.com/databases/databases-for-postgresql/create). Input your total consumption across two data members into the calculator. This is roughly double the size of your data because your data is replicated to both members. For example, 5 GB of disk and 1 GB of RAM across two data members would be priced at 10 GB of disk and 2 GB of RAM respectively. 

## Backups pricing
{: #pricing-backup}

You receive your total disk space purchased, per database, in free backup storage. For example, in a given month, if you have a {{site.data.keyword.databases-for-postgresql}} deployment that has 20 GB of disk per member, and has three data members, you receive 60 GB of backup storage free for that month. If your backup storage utilization is greater than 60 GB for the month (in this scenario), you are charged an overage of $0.03/month per gigabyte. 

By default, {{site.data.keyword.databases-for}} provides a daily backup that is stored for 30 days. These backups, and any on-demand backups you make, all count toward the above allocation.

In the above example, if your database contains 2 GB of data and you have not taken any on-demand backups, then your total backup size is 2 GB x 30 = 60 GB. Your backup costs are nil.

If your database contains 15 GB of data and you have not taken any on-demand backups, then your total backup size is 15 GB x 30 = 450 GB. In this scenario, your backup costs are (450 GB - 60 GB) * 0.03 = $11.7 per month.

Most deployments will not ever go over the allotted credit.

## Scaling per member
{: #scaling-member}

{{site.data.keyword.databases-for-postgresql}} deployments have minimum and maximum allocation for disk and RAM as shown. Scaling deployments through the API/CLI provides more granularity and also allows a user to scale a database instance up to 4 TB of disk per member.

| Resource | Minimum | Maximum | Scaling Granularity (API/CLI) |
| ---------- | ----- | ----- | ------- |
| Disk | 5 GB per member | 4 TB per member | 1024 MB per member |
| RAM | 1 GB per member | 112 GB per member | 128 MB per member |
| CPU (if enabled) | 0.5 vCPUs per member | 28 vCPUs per member| 2 CPU per member |
{: caption="Per Member Scaling Limits" caption-side="top"}
