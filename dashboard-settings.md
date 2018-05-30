---

Copyright:
  years: 2018
lastupdated: "2018-05-30"
---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}

# Settings
Manage your ICD for PostgreSQL service through these available settings.

## Scaling

Resources that are unable to be scaled are greyed out. They are present for informational purposes only. 

Scaling operations can cause downtime.

Billing is based on the _total_ amount of resources allocated to the service. 

### Storage
The default disk space for a ICD PostgreSQL serivce is ____GB. After that, disk space allocated based on the size of your data. As you store more data in the service the amount of disk space will increase automatically. Your data is replicated across two data members in the PostgreSQL cluster, so the amount of space your usage reflects is roughly twice the size of your data. Billing is based on the total amount of disk space allocated to your service.

### Memory

The default size of memory allocated to your service is ____MB. If you find that your queries and database activity suffers from performance issues due to a lack of memory, you may scale the amount of RAM allocated to your service. You scale the amount of RAM on the _Settings_ pane of the service.

Adjust the slide to raise or lower the memory allocated to your service. Click **Scale** button to trigger the rescaling and return to the dashboard overview.

When the scaling is complete the Deployment Details pane updates to show the current usage and the new values for the available storage and memory. 

### Scaling from the API





