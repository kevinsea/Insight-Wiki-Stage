# Insight and Data Providers #

Most of the features in Insight work with the .NET System.Data interfaces and have no dependencies on the data provider.

However, there are a few features that only work with certain data providers.

## SQL Server ##
The following features require SQL Server as the data provider:

* Stored Procedure parameter detection (see [[Stored Procedures vs SQL Text]])
* ConnectionStringSettings extensions (see [[Opening Connections]])
* Xml Parameters and Results (see [[Xml Parameters and Results]])