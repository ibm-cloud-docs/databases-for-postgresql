---

Copyright:
  years: 2018
lastupdated: "2018-05-14"
---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}

# Architecture

IBM Cloud Databases are managed DbaaS systems that run in containers orchestrated by Kubernetes. It's fully integrated into the IBM Cloud ecosystem. The database and it's asociated infrastructure all run in IBM Cloud.

## High-level Overview

### Databases and Portals
Each member of a Kubernetes cluster manages contianers in pods. There are two types of pods:
- Pods for the database application containers
- Pods for the portal containers used to access the database. 

Each of these types of pods have all the necessary containers to run and maintain their associated operations and configurations.
The database pods contain:
- the database application containers -- actual database storage and application
- the management containers -- scripts, backup management, replication and recovery management, scaling and other mangaged service infrastructure
- the telemetry and metrics containers -- logging, usage monitoring, and other container/database monitoring

The portal pods contain:
- the HAProxy containers -- controls the HTTP connections, TLS/SSL termination, authentication, and load-balancing
- the management containers -- scripts, whitelists, fail-over monitoring and special portal configurations
- the telemetry and metrics containers -- connection and memory monitoring

### Storare
All storage for ICD is provided on IBM Cloud baremetal/SSDs/VMWare/whatever. Each host gets provisioned into very large storage chunks. This storage is then further partitioned into volumes that run Kubernetes clusters which in turn dynamically allocates the storage to a cluster's pods.


### Replication and High-Availability
Database applications in ICD all come configured with replication across the Kubernetes cluster. These clusters are spread over the regions' availability zones. Link to doc about regions and AZs.

Cross-regional replication is not an available feature of ICD at this time.


## Monitoring
Monitoring is provided by telegraf and Marmot ctmonitor running along side the database containers in each pod. Each cluster then has a service that takes the collected database, container, pod, and cluster metics and routes them to the internal alert system. All relevant container and database application monitoring is collected and sent through telegraf to the IBM Cloud Monitoring Service.


## Access Management
IAM ---wheee!


## Logs
All logs from IBM Cloud Database services are pushed into the IBM Cloud Log Analysis service. This includes:
- Database application logs from each container running inside pods
- HAProxy logs from each container running inside pods

