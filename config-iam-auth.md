---

copyright:
  years: 2021
lastupdated: "2021-06-16"

keywords: IBM Cloud, databases, Satellite, ICD, IAM authorization

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

{:shortdesc: .shortdesc}
{:screen: .screen}  
{:codeblock: .codeblock}  
{:pre: .pre}
{:tip: .tip}
{:note: .note}
{:external: target="_blank" .external}

# Configuring IAM Authorizations
{: #tutorial-pgsql-iam}

To create a storage configuration for a newly created IBM Cloud Databases for PostgreSQL enabled by IBM Cloud Satellite service location, you need to configure IAM authorizations.
{: shortdesc}

## Configure IAM Authorizations
{: #cd-postgresql-iamauth}

- Configure your IAM Authorizations under the **Manage** tab.

- Choose the **Authorizations** tab from the lefthand menu.
- Click the **create** button to create an authorization to allow a service instance access to another service instance.

## Grant a service authorization
{: #cd-postgresql-serviceauth}

The source service is the service that is granted access to the target service. The roles that you select define the level of access for this service. The target service is the service you are granting permission to be accessed by the source service based on the assigned roles.

- In the **Source Service** field, select **Databases for PostgreSQL development**.
- In the **Target Service** field, select **Satellite**.
- Select all options:
  - **Satellite Cluster Creator** 
  - **Satellite Link Administrator** 
  - **Satellite Link Source Access Controller** 
 - Then **Authorize**.
