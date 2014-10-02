**Insight.Database** is a fast, lightweight, (and dare we say awesome) micro-orm for .NET.

It's available as a [NuGet Package](http://www.nuget.org/packages/Insight.Database/).

## Why You Want Insight.Database ##
It **just works**...

- Without a lot of effort or configuration.

It's **fast**...

- Mappers are generated IL code, not runtime reflection.
- Mappings are cached.
- Optimized for SequentialAccess.
- Async operations are first-class operations.

It's **easy**...

- Automatically maps fields/properties to parameters.
- Automatically maps columns to fields/properties.
- Automatic one-to-one, one-to-many, many-to-many, and multi-recordset support.
- Wire up an interface definition right to database calls.
- Generic extension methods automatically handle type safety for you.

It's **flexible**...

- It works with all of the common databases and tools.
- It supports structured, production quality coding.
- It **also** supports ad-hoc, one-off, typeless, get-it-done coding.
- Use stored procedures or inline SQL.
- Use POCOs or dynamics.
- Awesome type conversion support. Never worry about mismatches between property types and column types.
- Override mappings through attributes or configuration. Or not.
- Extend the system with your own readers if you need to. Or not.

It's **advanced**...

- Support for BulkCopy (most providers)
- Support for cloud providers through ReliableConnection
- Support for optimistic concurrency through OptimisticConnection

It's **documented**...

- Take that, most of you other open-source projects!

## Before You Begin ##

The documentation below is written so that you understand Insight from the ground up. Regardless of your coding style, If you read it in order, you will find ways to make your database code simpler and easier.

Of course, what I **really** want you to do is to switch over to [[Auto Interface Implementation]]. So please read at least that far before you get too excited and start coding.

## A Quick Tour ##

* [[A Quick Tour]] - see how easy it is!

## Getting Started ##

* [[Installing Insight]] - just get it from NuGet.
* [[What is New in 5.0]] - composite keys, deep parameter binding.
* [[What is New in 4.0]] - one-to-many mappings and other structural goodness.
* [[Design Goals]] - why does this library work this way?

## Executing Queries ##

* [[Getting A Connection]] - start at the beginning.
* [[Opening Connections]] - no, you won't really need this.
* [[Auto-Open]] - be done with `using` statements and lifetime management.
* [[Executing SQL Commands]] - just the basics.
* [[Querying for Objects]] - getting objects back with no work.

## Common Query Scenarios ##

* [[Identity Inserts]] - getting IDs and other data back from the database and merged into your object.
* [[Lists of Objects as Parameters]] - sending a bunch of objects to a database has never been easier.
* [[Transactions]] - executing more than one method in a transaction.
* [[Optimistic Concurrency]] - being cautious about changes to records.
* [[Output Parameters]] - how to access Output Parameters from a stored procedure.

## Object Mapping ##

* [[Query Parameter Mapping]] - how Insight maps objects to query parameters.
* [[Mapping Results To Objects]] - how Insight maps results to objects.
* [[Customizing Object Mapping]] - how to override the default mapping rules.

## Specifying Result Structures ##

* [[Specifying Result Structures]] - convert recordsets into complex object structures.
* [[Record Readers]] - how Insight reads individual records.
* [[Query Readers]] - how Insight reads structures.

## Doing it the Right Way ##

* [[Auto Interface Implementation]] - define an interface to your database. Let Insight do the rest.

## Nits and Gnats ##

* [[Common Method Parameters]] - the parameters for most of the extension methods.
* [[Manual Transformations]] - how to do your own transformations from data to objects.
* [[Custom Result Objects]] - some fancy tricks.

## Data Types ##
* [[Date-Based Data Types]] - how date-based types are handled.
* [[Enums and Mapping]] - the rules for mapping enums to/from your database.
* [[Linq Binary]] - equivalent to a byte array when sending in as a parameter.
* [[Object Serialization]] - how object fields are serialized to/from string values.
* [[Xml Parameters and Results]] - XmlDocument, xml columns, objects and strings all play nicely.

## Dynamic Objects and Queries ##
* [[Dynamic Database Calls]] - make your SQL procs just look like methods on your Connection.
* [[Dynamic Objects]] - for when you don't want to bother making a class just for one query.
* [[FastExpando and the Borg Method]] - using FastExpando to assimilate objects and clean up some icky parameter code.
* [[FastExpando and Mutations]] - using FastExpando to clean up mappings between icky database and clean code.

## Performance & Speed ##
* [[Async Commands and Queries]] - just add Async and you're done.
* [[Bulk Copy with Objects]] - for when you have a bunch of work to do.
* [[Streaming Results Efficiently]] - how to minimize memory usage by operating on objects as they are streamed in
* [[ReliableConnection and Cloud Databases]] - easy retry logic in one line of code.

## Database Providers ##

* [[Insight and Data Providers]]
* [[SQL Server Provider]]
* [[DB2 Provider]]
* [[MySql Provider]]
* [[SQLite Provider]]
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
* [[Stored Procedures vs SQL Text]] - discusses the differences between using Stored Procedures and SQL Text with Insight.
* [[Creating Commands]] - how to create a command manually for reuse.
* [[Strong Named Assemblies]] - some notes on the signed assemblies in v3.2 and later.