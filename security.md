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

# Security and Compliance


## Protection Against Unauthorized Access

IBM Cloud Databases use the following methods to protect data in trasit or from storage.
- All IBM Cloud Databases use TLS/SSL encryption for data in transit. The current supported version of this encryption is TLS 1.2.
- Access to the Account, Management Console UI, and API is secured via IAM (Identity & Access Management)
- Access to the database is secured through the standard access controls provided by the database. These access controls are configured to require a valid set of database-level credentials which are obtainable only through prior access to the database or through our Management Console UI or API.
- All IBM Cloud Database storage is provided on encrypted volumes using the Linux Unified Keys Setup (LUKS).  

## Data Resilience

- Backups are included in the service. Backups reside in the same cloud storage location as the database service itself, and so are also encrypted.
- All IBM Cloud Databases are configured with replication, so the data exists with multiple copies and each copy resides on a different cluster and host. Where available, those clusters are also spread in different avalibility zones within the region where the service is deployed.

## GDPR

As a data processor our services are compliant with GDPR regluations regarding data processors. The IBM Data Processing Addendum is available here: https://www-05.ibm.com/support/operations/zz/en/dpa.html

## Terms

The IBM Privacy Policy: https://www.ibm.com/privacy/us/en/
The IBM Cloud Notices and Terms of Use: https://console.bluemix.net/docs/overview/terms-of-use/notices.html#notices


