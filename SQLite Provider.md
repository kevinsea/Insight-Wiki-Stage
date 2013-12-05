# SQLite Provider #

## Requirements ##

Insight.Database v3.1.4 and later have been tested with [System.Data.SQLite](http://www.nuget.org/packages/System.Data.SQLite/). Just install System.Data.SQLite from NuGet and use Insight.Database with your SQLiteConnections. 

Since SQLite doesn't support stored procedures, Insight.Database can handle all of the features without needing code specific to SQLite. So you don't need to install an Insight.Database.Provider for it.

## Known Issues ##

SQLite does not support the following features:

* Stored Procedures
* Table-Valued Parameters
* XML Columns
* Bulk Copy
