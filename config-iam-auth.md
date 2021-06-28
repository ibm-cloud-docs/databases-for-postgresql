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
<!--  {:step: data-tutorial-type='step'} Apply to steps for automatic numbering -->

<!-- The title of your tutorial should be in active voice and and start with a verb. If you include product names, makes sure to use the non-trademarked short version conref. -->
<!-- Make sure each H1/H2/H3/etc. heading is _unique_ to your tutorial by adding a short but human-readable identifier. For example, instead of just "#overview", use "#cd-kube-overview" -->

# Configuring IAM Authorizations
{: #tutorial-pgsql-iam}
<!--{: toc-content-type="tutorial"}  Always use this value -->
<!--{: toc-completion-time="10m"} <!-- Use same value from completion-time metadata above-->

<!-- The short description should be a single, concise paragraph that contains one or two sentences and no more than 50 words. Briefly mention what the user's learning goal is and include the following SEO keywords in the title short description: IBM Cloud, ServiceName, tutorial.--> 

In this tutorial, you will learn how to create a storage configuration for a newly created IBM Cloud Databases for PostgreSQL enabled by IBM Cloud Satellite service location.
{: shortdesc}

<!-- It's recommended to include an architectural diagram that shows how the services that are used in this tutorial interact. SVG is the recommended format. If you include a diagram, include a brief text-based description of the workflow shown in the diagram, using active voice to describe the workflow. This makes the content more searchable and improves accessibility. -->

<!--![Architectural diagram](images/image.svg)
{: figure caption="Figure 1. A diagram that shows the architecture for my tutorial."}

<!-- The pipeline that you create has the following architecture:
1. Workflow step 1
1. Workflow step 2
1. Workflow step 3
1. Workflow step 4-->

## Before you begin
{: #postgres-sat-prereqs}

* To get started with storage configuration, you will need to have completed the following steps for your location: 
  * Set up a Satellite service.
  * Create a satellite location. 
  * Attach hosts to your location.
  * Create storage configuration.

To review any of the above steps, refer to the [Getting Started with IBM Cloud Databases for PostgreSQL enabled by IBM Cloud Satellite documentation](docs/howto-sat-location.md).
{: note}

## Configure IAM Authorizations
{: #cd-postgresql-iamauth}

<!-- Introduce each major step with a description of what it will accomplish. If there are sequential substeps, use an ordered list for each substep. Don't include the step number. -->

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
