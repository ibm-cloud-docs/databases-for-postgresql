---

Copyright:
  years: 2017,2018
lastupdated: "2018-05-14"
---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}

# Pricing

## Base Configuration
An ICD for PostgreSQL service is a cluster with two data nodes, each data node containing 1GB storage and 102MB of memory, with a total size of 2GB storage and 204MB of memory. The cost of the service is based on the total size of the service across these two dimentions.

The base configuration also includes three etcd arbiter capsules to manage replication, and two HAProxy capsules for managing connections and fail-over. The HAProxy capsules each have 64MB of memory. 

The base configuration also includes space for ____ of backups.

## Expansion Pricing
The ICD for PostgreSQL is adaptable to your use-case, with options to expand storage (automatic), RAM (manual), and set backup policies that suit your needs.
Here are the finalized structures for what we are charging. To see pricing in your local currency please reference the IBM Cloud Catalog page.

PID # IBM Cloud Databases for PostgreSQL

Parts # Public Compute (This only surfaces on the IBM catalog Page)
CC:
Storage per Hour ($ per GB/Hour)
RAM per Hour ($ per GB/Hour)
Backup Storage per Month ($ per GB/Month)

Parts # Isolated Compute (This only surfaces on the IBM catalog page)
CC:
Storage per Hour ($ per GB/Hour)
RAM per Hour ($ per GB/Hour)
Virtual Cores per Hour (Virtual Processor Core/Hour)
Backup Storage per Month ($ per GB/Month)