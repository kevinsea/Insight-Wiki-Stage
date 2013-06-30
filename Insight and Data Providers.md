# Insight and Data Providers #

The big change in v3.0 is the implementation of a provider model for databases. Providers allow the core Insight.Database library to work without any dependencies on a particular database. Any database-specific code is implemented in a separate provider.

Since .NET ships with a few database clients, the following providers are built-in and work automatically:

* [[SQL Server Provider]] - all features
* [[ODBC Provider]] - common features, no TVPs, no XML
* [[OLEDB Provider]] - common features, no TVPs, no XML 

Other providers require additional assemblies or dependencies, so they are shipped separately from the core Insight.Database install:

* [[MySql Provider]] - common features, no BulkCopy, no TVPs, no XML
* [[Oracle Provider]] - all features, a few known issues
* [[Oracle Managed Provider]] - common features, no BulkCopy, no TVPs, no XML
* [[MiniProfiler Provider]] - all features