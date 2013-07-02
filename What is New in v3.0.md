# What's New in v3.0 #

The big change in v3.0 is the implementation of a provider model for databases. Providers allow the core Insight.Database library to work without any dependencies on a particular database. Any database-specific code is implemented in a separate provider.

Since .NET ships with a few database clients, the following providers are built-in and work automatically:

* [[SQL Server Provider]]
* [[ODBC Provider]]
* [[OLEDB Provider]]

Other providers require additional assemblies or dependencies, so they are shipped separately from the core Insight.Database install:

* [[MySql Provider]]
* [[Oracle Provider]]
* [[Oracle Managed Provider]]
* [[PostgreSQL Provider]]
* [[MiniProfiler Provider]]