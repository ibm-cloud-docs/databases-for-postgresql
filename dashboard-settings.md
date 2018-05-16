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

# Settings

## Usage
The _Overview_ page of yhe service contains a quick summary of the resource usage of the service. To get a breakdown of the resources allocated and used by your service, got to the _Settings_.


## Scaling

### Storage
Disk usage for your data is allocated to the size of your data on disk. As you store more data in the service the amount of disk space will increase automatically. Your data is replicated across two data members in the PostgreSQL cluster, so the amount of space your usage reflects is roughly twice the saize of your data. Billing is based on the total amount of disk space allocated to your service.
Max limit is present; contact support for more scaling
You will not be able to scale the amount of storage allocated your service below the current size of your service.

### Memory
The default size of memory allocated to your service is ____MB. If you find that your queries and database activity suffers from performance issues due to a lack of memory, you may scale the amount of RAM allocated to your service. You scale the amount of RAM on the _Settings_ pane of the service.
Adjust the slider to raise or lower the storage allocated to your service.
Adjust the slide to raise or lower the memory allocated to your service.
Resources that are unable to be scaled are greyed out. They are present for informational purposes only. 
Scaling operations can cause downtime (?)
Click Scale Deployment to trigger the rescaling and return to the dashboard overview.

When the scaling is complete the Deployment Details pane updates to show the current usage and the new values for the available storage and memory. RAM is scaled on both members of your cluster and billing is based on the total amount of RAM allocated to the service.
Max limit is present; contact support for more scaling
You will not be able to reduce memory below ___MB.

### Scaling from the API





