Insight.Database is a fast, lightweight, (and dare we say awesome) micro-orm for .NET.

Insight.Database lets you call your database with almost no code, and makes it easy to send objects to your database and get them back. It's available under MS Public License, but we prefer if you use the BuyAFriendABeerAndTellThemAboutInsight License.

**v3.0 is now in NuGet! Now with support for more database servers! See the [[Change Log]] for what's new!**

Follow [@jonwagnerdotcom](http://twitter.com/#!jonwagnerdotcom) for latest updates on this library or [code.jonwagner.com](http://code.jonwagner.com) for more detailed writeups.

# Why You Want This #
- It just works. 
- Without a lot of effort or configuration. 
- It's **fast**.
- It supports structured, production quality coding.
- It **also** supports ad-hoc, one-off, typeless, get-it-done coding.
- Put your database behind an interface with no effort.
- Objects are mapped automatically, but still give you control if you need it.
- Async queries are first-class operations.
- Deserializing multiple recordsets are first-class operations.
- ReliableConnection fully supports cloud providers such as SQL Azure.

## A Quick Tour ##

* [[A Quick Tour]] - see how easy it is!

## Getting Started ##
* [[Design Goals]] - why does this library work this way?
* [[Installing Insight]] - just get it from NuGet.
* [[What is New in v2.0]] - some WayCool(tm) stuff.
* [[What is New in v3.0]] - providers, providers, providers.

## Executing Queries ##
* [[Auto-Open]] - be done with `using` statements and lifetime management.
* [[Opening Connections]] - in case you want to manually open connections.
* [[Executing SQL Commands]] - just the basics.
* [[Auto Interface Implementation]] - just about the coolest thing ever.
* [[Querying for Objects]] - getting objects back with no work.
* [[Common Method Parameters]] - the parameters for most of the extension methods.
* [[Lists of Objects as Parameters]] - sending a bunch of objects to a database has never been easier.
* [[Identity Inserts]] - getting IDs and other data back from the database and merged into your object.
* [[Transactions]] - executing more than one method in a transaction.

## Controlling Object Mapping ##
* [[Query Parameter Mapping]] - how Insight maps objects to query parameters.
* [[Mapping Results To Objects]] - how Insight maps results to objects.
* [[Customizing Object Mapping]] - how to override the default mapping rules.
* [[Object Graphs]] - how to deserialize an object graph from a record set.
* [[Multiple Result Sets]] - how to easily map queries with multiple result sets.
* [[Output Parameters]] - how to access Output Parameters from a stored procedure.
* [[Manual Transformations]] - how to do your own transformations from data to objects.
* [[Enums and Mapping]] - the rules for mapping enums to/from your database.
* [[Date-Based Data Types]] - how date-based types are handled.

## Dynamic Objects and Queries ##
* [[Dynamic Database Calls]] - make your SQL procs just look like methods on your Connection.
* [[Dynamic Objects]] - for when you don't want to bother making a class just for one query.
* [[FastExpando and the Borg Method]] - using FastExpando to assimilate objects and clean up some icky parameter code.
* [[FastExpando and Mutations]] - using FastExpando to clean up mappings between icky database and clean code.

## Data Types ##
* [[Xml Parameters and Results]] - XmlDocument, xml columns, objects and strings all play nicely.
* [[Linq Binary]] - equivalent to a byte array when sending in as a parameter.

## Performance & Speed ##
* [[Async Commands and Queries]] - just add Async and you're done.
* [[Bulk Copy with Objects]] - for when you have a bunch of work to do.
* [[Streaming Results Efficiently]] - how to minimize memory usage by operating on objects as they are streamed in
* [[ReliableConnection and Cloud Databases]] - easy retry logic in one line of code.

## Database Providers ##

* [[SQL Server Provider]]
* [[DB2 Provider]]
* [[MySql Provider]]
* [[Oracle Provider]]
* [[Oracle Managed Provider]]
* [[PostgreSQL Provider]]
* [[ODBC Provider]]
* [[OLEDB Provider]]

## Test & Profiling Framework Providers ##

* [[Glimpse Provider]]
* [[MiniProfiler Provider]]

## Revision History ##
* [[Change Log]]

## Other Stuff ##
* [[Code Snippets]] - install these into Visual Studio to get instant repository code.
* [[Stored Procedures vs SQL Text]] - discusses the differences between using Stored Procedures and SQL Text with Insight.
* [[Creating Commands]] - how to create a command manually for reuse.
* [[Insight v Dapper]] - Dapper was the inspiration for parts of Insight, but now they are totally different.
