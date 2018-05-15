# Security and Compliance


## Protection Against Unauthorized Access

IBM Cloud Databases use the following methods to protect data in trasit or from storage.
- All IBM Cloud Databases use TLS/SSL encryption for data in transit. The current supported version of this encryption is TLS 1.2.
- Certificates line (if necessary, or wanted)
- Soflayer/IBM Cloud data center security
- Access to the Account, Management Console UI, and API is secured via IAM (Identity & Access Management)
- Access to the database is secured through the standard access controls provided by the database. These access controls are configured to require a valid set of database-level credentials which are obtainable only through prior access to the database or through our Management Console UI or API.
- Access by IBM Cloud support policies
- Access logs and monitoring if such things are available/wanted
- Whitelists -- when it is a feature
- All IBM Cloud Database storage is provided on encrypted volumes using the Linux Unified Keys Setup (LUKS).  
- Isolated compute environments have the ability to BYOK (?)

## Data Resilience

- Backups reside in the same cloud storage location as the database service itself, and so are also encrypted.
- All IBM Cloud Databases are configured with replication, so the data exists with multiple copies and each copy resides on a different cluster and host. Where available, those clusters are also spread in different avalibility zones within the region where the service is deployed.

## GDPR

As a data processor our services are compliant with GDPR regluations regarding data processors.
All ICD services provisioned in all/some/these regions are compliant with GDPR. 
The following specifics around not sharing any of the data you store are here. 
The following specifics around internal IBM support for your data and databases not leaving the EU are here.
The following specifics with what happens to any data (including any data you have that may be PII) when you delete databases, services, or close your account will go here.

## HIPPA
TBA

## SOC 2
TBA


## Terms of Service

1. Customer agrees to Terms of Service on Compose.com https://ibm.box.com/s/5mqyi8uaobshbv34ji6c20qxozs5gwn5 (this agreement would contain the following):
    1. Data Sheet is embedded in the terms: contains info on data collected and data processed
    2. In the terms, there is a link to the IBM DPA: https://www-05.ibm.com/support/operations/zz/en/dpa.html
The box document would essentially replace the existing compose Terms, one outstanding question for Debbie is how we get customers to agree to those, I believe she has already thought through that I can't remember the status. Customers would choose to 'accept' these terms which would be consent. But there isn't a formally signed document we plan to present to them. 
