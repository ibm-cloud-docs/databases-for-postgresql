---

Copyright:
  years: 2018
lastupdated: "2018-06-18"
---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}
{:tip: .tip}

# Settings

Manage your ICD for PostgreSQL service through these available settings.

## Scaling

The _Scale Resources_ panel shows the current size and resource allocation for your service. You can manage resources available to your service by adjusting the individual resources. Resources that can be scaled will have a slider. Adjust the slider to raise or lower the resource allocated to your service. Click **Scale** button to trigger the rescaling and return to the dashboard overview.

Resources that are unable to be scaled are greyed out. They are present for informational purposes only.
{: .tip} 

Scaling operations can cause downtime. When the scaling is complete the Deployment Details pane updates to show the current usage and the new values for the available storage and memory. 

Billing is based on the _total_ amount of resources allocated to the service. 

### Storage
Disk space allocated based on the size of your data. As you store more data in the service the amount of disk space will increase automatically. Your data is replicated across two data containers in the PostgreSQL cluster, so the amount of space your usage reflects is roughly twice the size of your data. Billing is based on the total amount of disk space allocated to your service.

### Memory

If you find that your queries and database activity suffers from performance issues due to a lack of memory, you may scale the amount of RAM allocated to your service. Your PostgreSQL service runs with two containers in a cluster, so the amount of memory you add will be added to both containers. Billing is based on the total amount of memory allocated to your service.







