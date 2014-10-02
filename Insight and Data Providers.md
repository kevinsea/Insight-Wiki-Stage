# Insight and Data Providers #

The big change in v3.0 is the implementation of a provider model for databases. Providers allow the core Insight.Database library to work without any dependencies on a particular database. Any database-specific code is implemented in a separate provider.

Since .NET ships with a few database clients, the following providers are built-in and work automatically:

* [[SQL Server Provider]] - all features
* [[ODBC Provider]] - common features, no TVPs, no XML
* [[OLEDB Provider]] - common features, no TVPs, no XML 
* MS Access - use the ODBC or OLEDB Provider
* [[SQLite Provider]] - common features work with the default provider

Other providers require additional assemblies or dependencies, so they are shipped separately from the core Insight.Database install:

* [[DB2 Provider]] - all features except TVPs
* [[MySql Provider]] - common features, no BulkCopy, no TVPs, no XML
* [[Oracle Provider]] - all features, a few known issues
* [[Oracle Managed Provider]] - common features, no BulkCopy, no TVPs, no XML
* [[Sybase ASE Provider]] - all features except TVPs
* [[PostgreSQL Provider]] - common features, no TVPs
* [[Glimpse Provider]] - all features of the underlying database
* [[MiniProfiler Provider]] - all features of the underlying database

If you don't install a provider for your database, Insight 3.1.4 and later will use a default provider that will give you some basic functionality.

## (Possibly) Important Note ##

Currently (v3.x), Insight assumes that you will only use one provider for a given CommandText or ProcName. If you issue the same command against different providers, Insight will reuse the cached plan from the first provider. This probably won't happen to you, but if it does, open an issue and we'll take the time to make a cache per provider.