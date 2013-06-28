# Insight and Data Providers #

The big change in v3.0 is the implementation of a provider model for databases. Providers allow the core Insight.Database library to work without any dependencies on a particular database. Any database-specific code is implemented in a separate provider.

Since .NET ships with a few database clients, the following providers are built-in and work automatically:

* [[SQL Server Provider]] - all features
* [[ODBC Provider]] - common features, no TVPs or XML
* [[OLEDB Provider]] - common features, no TVPs or XML 

Other providers require additional assemblies or dependencies, so they are shipped separately from the core Insight.Database install:

* [[Oracle Provider]] - all features, a few known issues
* [[MiniProfiler Provider]] - all features