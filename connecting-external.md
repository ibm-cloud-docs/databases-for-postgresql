---
copyright:
  years: 2017,2018
lastupdated: "2017-07-18"
---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}

# Connecting an external application
{: #connecting-external-app}




## Getting Connection Strings

A page (./work-with-connection-strings)

The _Service Credentials_ panel provides connection strings as a JSON object. 
Retrieving connection strings from the CLI returns a table or JSON.

{{site.data.keyword.databases-for-postgresql}} provides connection strings specifically for drivers and applications.
A [table](./working-connection-strings#the-postgresql-section) with a breakdown of all the connection information.

## Connecting with a language's driver

Postgres has a vast array of language drivers. The table covers a few of the most common.

Language|Driver|Examples
----------|-----------
PHP|`pgsql`|[Link](http://php.net/manual/en/pgsql.examples-basic.php)
Ruby|`ruby-pg`|[Link](https://bitbucket.org/ged/ruby-pg/wiki/Home)
Ruby on Rails|Rails|[Rails Guide](http://edgeguides.rubyonrails.org/configuring.html#configuring-a-postgresql-database)
Python|`Psycopg2`|[Link](https://wiki.postgresql.org/wiki/Psycopg2_Tutorial)
C#|`ODBC`|[Link](https://wiki.postgresql.org/wiki/Using_Microsoft_.NET_with_the_PostgreSQL_Database_Server_via_ODBC)
Go|`pq`|[Link](https://godoc.org/github.com/lib/pq)
Node|`node-postgres`|[Link](https://github.com/brianc/node-postgres/wiki/Example)

## Driver TLS and self-signed certificate support









 
