---
copyright:
  years: 2019, 2021
lastupdated: "2021-08-23"

keywords: postgresql, databases, icu

subcollection: databases-for-postgresql

---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}
{:tip: .tip}

# PostgreSQL ICU Support
{: #postgres-icu}
[ICU](http://site.icu-project.org/) (International Components for Unicode) uses C/C++ and Java libraries to provide Unicode and Globalization internationalization and localization facilities, including collation. ICU provides stability to your database by preventing index corruption caused by operating system collation changes. 

{{site.data.keyword.databases-for-postgresql_full}} supports ICU. To use ICU, your deployment needs to be built explicitly with ICU support. ICU-based collations are offered alongside the `libc` collations (which use the locales provided by the operating system C library), so building with ICU support does not remove `libc` collation support. Consult the [PostreSQL Collation Support documentation](http://www.postgresql.org/docs/10/static/collation.html) for details on selecting an ICU versus `libc` collation.

## Enabling ICU
{: #enabling-icu}
While ICU collations are available, {{site.data.keyword.databases-for-postgresql_full}} are the same. 

To deploy with support for the ICU library, the ICU4C package must be installed by appending the `--with-icu` flag. 

To enable ICU support on your {{site.data.keyword.databases-for-postgresql_full}} deployment, append your commands with `-x-icu` flag.
