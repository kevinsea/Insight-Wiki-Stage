# Sybase ASE Provider #

## Requirements ##

Insight.Database works with Sybase ASE. You must first install the Sybase ASE provider for .NET (Sybase.AdoNet4.AseClient) and add it to your project. 

## Initializing the SybaseAseInsightDbProvider ##

Before you can use Insight.Database with Sybase, you must first install and register the provider in your application. 

1. Install the Insight.Database.Providers.SybaseAse package from NuGet.
2. Add a reference to Sybase.AdoNet4.AseClient to your project.

## Known Issues ##

* Table-Valued Parameters are not supported.
