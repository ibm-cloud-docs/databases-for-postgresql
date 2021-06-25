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
{:java: .ph data-hd-programlang='java'}
{:javascript: .ph data-hd-programlang='javascript'}
{:swift: .ph data-hd-programlang='swift'}
{:curl: .ph data-hd-programlang='curl'}
{:video: .video}
{:step: data-tutorial-type='step'}
{:tutorial: data-hd-content-type='tutorial'}
{:ui: .ph data-hd-interface='ui'}
{:cli: .ph data-hd-interface='cli'}
{:api: .ph data-hd-interface='api'}

# Getting started
{: #getting-started}
{: toc-content-type="tutorial"}
{: toc-completion-time="15m"}

With IBM Cloud Databases for PostgreSQL enabled by IBM Cloud Satellite, you can choose a location to represent a data center that you fill with your own infrastructure resources and start running IBM Cloud™ Databases on your own infrastructure.
{: shortdesc}

## Before you begin
{: #before-start}
- You need to have an [{{site.data.keyword.cloud_notm}} account](https://cloud.ibm.com/registration){:new_window}.
- You will also need a {{site.data.keyword.databases-for-postgresql}} deployment. You can provision one from the [{{site.data.keyword.cloud_notm}} catalog](https://cloud.ibm.com/catalog/services/databases-for-postgresql). Give your deployment a memorable name that appears in your account's Resource List.
- [Set the Admin Password](/docs/databases-for-postgresql?topic=databases-for-postgresql-admin-password) for your deployment.

Before setting up your satellite location, see below regarding IBM Cloud Databases for PostgreSQL enabled by IBM Cloud Satellite service limitations. 
{: .note}

## Setting Up a Satellite Service 
{: step}

- Log in to your [{{site.data.keyword.cloud_notm}} account](https://cloud.ibm.com/registration){:new_window}.
- Create a new Satellite location.
- Create and attach Satellite location's AWS hosts that will be assigned the control plane.
- Create and attach AWS hosts for the Satellite location's data plane.
- Create a Satellite location storage configuration.

## Service Limitations
IBM Cloud Databases for PostgreSQL enabled by IBM Cloud Satellite service limitations are as follows:
- AWS support is dependent upon Satellite's AWS block storage support, specifically [Amazon Elastic Block Store (EBS)](https://aws.amazon.com/ebs/).
- Azure is not available.
- Google Cloud is not available.
- Satellite location is limited to `us-east (wdc)`.
- IBM Cloud Databases for PostgreSQL enabled by IBM Cloud Satellite's control plane in `eu-de` is `EU-managed`; therefore, the Satellite service itself is `EU managed`. ICD does not support `non-EU managed` locations in FRA.
- IBM Cloud Databases for PostgreSQL enabled by IBM Cloud Satellite has a limitation of 20 locations per IBM Cloud MZR, per account. For more information, see [Satellite usage requirements](https://test.cloud.ibm.com/docs/satellite?topic=satellite-requirements).
- IBM Cloud Databases for PostgreSQL enabled by IBM Cloud Satellite does not support Amazon Virtual Private Cloud (VPC).
- There is no guaranteed SLA.
- There is no SOC 2 support.
- There is no HIPAA support.
- There is no PCI support.
- There is no FS Cloud support.
- There is no FedRAMP support.
- There is no Disaster Recovery support.
- There is no disk encryption support. 
- There is no BYOK for Disk encryption support.
- There is no read-only replica support.
