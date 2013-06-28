# Oracle Managed Provider #

Oracle now has a fully-managed version of ODP.NET. It's easier to install and manage, but it has some limitations.

## ODP.NET Managed Driver ##

To install the ODP.NET managed driver:

1. Install the ODP.NET.MANAGED package from NuGet.

## Initializing the OracleInsightDbProvider ##

Before you can use Insight.Database with Oracle, you must first install and register the provider in your application. The provider allows Insight to handle some of the wacky things needed to get ODP.NET to work.

1. Install the Insight.Database.Providers.OracleManaged package from NuGet.
2. Call OracleInsightDbProvider.RegisterProvider(). 

## Limitations of the ODP.NET Managed Driver ##

ODP.NET does not support the following Oracle features:

* XML data types
* Bulk Copy
* Table-Valued Parameters or Custom Types

## Helpful Hints ##

See [[Oracle Provider]] for some helpful hints for using Oracle with .NET and Insight.Database.