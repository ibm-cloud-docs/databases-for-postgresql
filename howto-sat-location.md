---

copyright:
  years: 2021
lastupdated: "2021-06-16"

keywords: IBM Cloud, databases, Satellite, ICD, get started

subcollection: databases-for-postgresql

content-type: tutorial
services: databases-for-postgresql
completion-time: 15m

---

{:codeblock: .codeblock}
{:screen: .screen}
{:download: .download}
{:external: target="_blank" .external}
{:faq: data-hd-content-type='faq'}
{:gif: data-image-type='gif'}
{:important: .important}
{:note: .note}
{:pre: .pre}
{:tip: .tip}
{:preview: .preview}
{:deprecated: .deprecated}
{:beta: .beta}
{:term: .term}
{:shortdesc: .shortdesc}
{:script: data-hd-video='script'}
{:support: data-reuse='support'}
{:table: .aria-labeledby="caption"}
{:troubleshoot: data-hd-content-type='troubleshoot'}
{:help: data-hd-content-type='help'}
{:tsCauses: .tsCauses}
{:tsResolve: .tsResolve}
{:tsSymptoms: .tsSymptoms}
{:curl: .ph data-hd-programlang='curl'}
{:step: data-tutorial-type='step'}
{:tutorial: data-hd-content-type='tutorial'}
{:ui: .ph data-hd-interface='ui'}
{:cli: .ph data-hd-interface='cli'}
{:api: .ph data-hd-interface='api'}

# IBM Cloud™ Databases for PostgreSQL enabled by IBM Cloud Satellite
{: #postgresql-satellite}

With IBM Cloud™ Databases for PostgreSQL enabled by IBM Cloud Satellite, you can deploy IBM Cloud Database instances into a Satellite location. The IBM Cloud Database service will then install an ICD Satellite service cluster in your Satellite location on which your database instances will be deployed.
IBM Cloud™ Databases for PostgreSQL enabled by IBM Cloud Satellite supports Satellite locations on [Amazon Web Services (AWS)](https://cloud.ibm.com/docs/satellite?topic=satellite-aws), as well as on your on-premises data centers that are attached to the IBM Cloud management location in Washington, DC. 
Before proceeding, you should refer to the [Satellite usage requirements](/docs/satellite?topic=satellite-requirements). 
{: shortdesc}

## Step 1: Create your Satellite location 
{: #postgresql-satellite-location}

**Before you begin**:
- You must be the account owner, or have the [administrator permissions](/docs/satellite?topic=satellite-iam#iam-roles-clusters) to the required services in Identity and Access Management (IAM).
- Log in to your [{{site.data.keyword.cloud_notm}} account](https://cloud.ibm.com/registration){:new_window}.
- Create a new [Satellite](/docs/satellite?topic=satellite-locations) location by selecting the **Manual setup** tab.
  - For management location, choose **Washington DC**.
  - If you intend to create your Satellite location on AWS, we recommend adjusting the **host zones** to AWS-default zone names: **zone-1**, **zone-2**, and **zone-3**.
  - Create and attach hosts to the Satellite hosts, either AWS or on-prem.
  - Assign the created hosts to the control plane. 

## Step 2: Prepare Satellite location for deployment
{: #postgresql-satellite-host}

We recommend you prepare your Satellite location before deploying Satellite service into it. 
- Attach additional hosts to the Satellite location
- Provide a block storage configuration

A Satellite host represents a compute machine of your own infrastructure, either on-premises or AWS. AWS support is dependent upon Satellite's AWS block storage support, specifically [Amazon Elastic Block Store (EBS)](docs/satellite?topic=satellite-config-storage-ebs). You can attach hosts to a location and then assign the hosts to run services such as clusters. To deploy an instance into your Satellite location, you have to provide three **8x32** and three **32x128** hosts.
{: shortdesc}

1.  **Attach**: Your machine becomes a host after you successfully [attach the host](#attach-hosts) to a Satellite location by running a registration script on the machine. Your machine must meet the [minimum host requirements](/docs/satellite?topic=satellite-host-reqs). For AWS-specific configurations, see [Manually adding AWS hosts to Satellite
](/docs/satellite?topic=satellite-aws#aws-host-attach). 
2.  **Assign**: The hosts in your Satellite location do not run any workloads until you assign them as compute capacity to the control plane. 
- Assign three 8x32 **AWS m5d.2xlarge** AWS hosts that will be assigned the Satellite control plane.
- If deploying with AWS, you must assign three 32x128 **AWS m5d.8xlarge** AWS hosts that will be assigned the Satellite data plane.
