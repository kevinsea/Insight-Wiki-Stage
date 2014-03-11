# MySql Provider #

The MySql Provider allows Insight.Database to work with your MySql database.

## Initializing the MySqlInsightDbProvider ##

Before you can use Insight.Database with MySql, you must first install and register the provider in your application. The provider allows Insight to handle some of the connection-specific database issues.

1. Install the Insight.Database.Providers.MySql package from NuGet.

## Known Issues ##

* Bulk Copy is not supported.
* Table Valued Parameters are not supported.
* XML columns are not supported.