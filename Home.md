Insight.Database is a fast, lightweight, (and dare we say awesome) micro-orm for .NET.

If you are thinking that you need something that is simple and just works for almost any use case that you can think of, Insight probably does it.

## Getting Started ##
* [[Installing Insight]]
* [[Opening Connections]]
* [[Executing SQL Commands]]
* [[Auto-Open]]
* [[Querying for Objects]]
* [[Dynamic Database Calls]
* [[Dynamic Objects]]
* [[Lists of Objects as Parameters]]
* [[Multiple Result Sets]]
* [[Async Commands and Queries]]
* [[Bulk Copy with Objects]]
* [[Xml Parameters and Results]]
* [[Overriding Mapping with ColumnAttribute]]
* [[Design Goals]]

## Revision History ##
* [[Change Log]]

## Important Stuff ##
* [[Insight and Data Providers]] - limitations due to certain data providers not named SQL Server
* [[Query Parameter Mapping]] - how Insight maps objects to query parameters
* [[Mapping Results To Objects]] - how Insight maps results to objects
* [[Common Method Parameters]] - the extra, hidden default parameters for most of the Insight extension methods
* [[Stored Procedures vs SQL Text]] - discusses the differences between using Stored Procedures and SQL Text with Insight

## Advanced & Other Stuff ##
* [[Creating Commands]] - how to create a command manually for reuse
* [[Manual Transformations]] - how to do your own transformations from data to objects
* [[Streaming Results Efficiently]] - how to minimize memory usage by operating on objects as they are streamed in
* [[Object Hierarchies]] - how to deserialize an object hierarchy from a record set
* [[FastExpando and the Borg Method]] - using FastExpando to assimilate objects and clean up some icky parameter code
* [[FastExpando and Mutations]] - using FastExpando to clean up mappings between icky database and clean code
* [[Linq Binary]] - equivalent to a byte array when sending in as a parameter
