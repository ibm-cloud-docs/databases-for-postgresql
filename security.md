# Security and Compliance


## Protection Against Unauthorized Access
IBM Cloud Databases use the following methods to protect data in trasit or from storage.
- All IBM Cloud Databases use TLS/SSL encryption for data in transit. The current supported version of this encryption is TLS 1.2.
- Certificates line (if necessary, or wanted)
- Soflayer/IBM Cloud data center security
- Account level access is handled with IAM, or other IBM Cloud account/role management. 
- Database level access is the responsibility of the service/database owner and handled through each database application's mechanisms 
- Access by IBM Cloud support policies
- Access logs and monitoring if such things are available/wanted
- Whitelists -- when it is a feature

## Data Resilience
- All IBM Cloud Database storage is provided on encrypted volumes using the Linux Unified Keys Setup (LUKS).  
- Isolated compute environments have the ability to BYOK (?)
- Backups reside in the same cloud storage loaction as the database service itself, and so are also encrypted.
- All IBM Cloud Databases are configured with replication, so the data exists with multiple copies and each copy resides on a different cluster and host. Where available, those clusters are also spread in different avalibility zones within the region where the service is deployed.

## GDPR
As a data processor our servieces are compliant with GDPR regluations regarding data processors.
All ICD services provisioned in all/some/these regions are compliant with GDPR. 
The following specifics around not sharing any of the data you store are here. 
The following specifics around internal IBM support for your data and databases not leaving the EU are here.
The following specifics with what happens to any data (including any data you have that may be PII) when you delete databases, services, or close your account will go here.

## HIPPA
TBA

## SOC 2
TBA


## Terms
[Here is an example link to offical SLAs](http://www-03.ibm.com/software/sla/sladb.nsf/sla/bm-7477-03) (for 'Compose For' services) . Might be preceded by a summary paragraph here.
