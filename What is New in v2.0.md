# What's New in v2.0 #

## Automatic Interface Implementation (v2.1) ##

See [[Auto Interface Implementation]] for details on these changes:

* **As** - extension methods added. Give Insight an interface and it will automatically bind it to your stored procedures for you.

## Simplified Transaction Syntax ##

See [[Transactions]] for details on these changes:

* **OpenWithTransaction** - extension methods added to open a connection with an automatically started transaction.

## Asynchronous Commands and Queries ##

See [[Async Commands and Queries]] for details on these changes:

* **ExecuteScalarAsync** - was added to return scalar values.
* **MergeAsync** - was added to allow you to manually merge results into existing objects.
* **QueryResultsAsync** - was added to query multiple recordsets.
* **ToListAsync** - was renamed from `ToList`
* **Full Async Mode** - In .NET 4.5, Insight uses `DbDataReader.ReadAsync` and `MoveNextAsync` when available, to reduce blocking when reading additional records from the stream.
* **CancellationToken Support** - Async methods now support passing in a cancellationToken.

## Mapping Overrides ##

See [[Customizing Object Mapping]] for details on these changes:

* **Column Attribute** - was added to let you override the mapping of a field to a database column.
* **ColumnMapping Class** - was added to let you have precise control for mapping object members to columns or parameters.

## Object Graphs ##

See [[Object Graphs]] for details on these changes:

* **DefaultGraphAttribute** - was added so you can specify the default Object Graph to use when deserializing a given type of object.
* **withGraph / withGraphs Parameters** - were added so you can programmatically override the deserializer's object graphs without having to muck with the generic parameters.

## Multiple Result Sets ##

See [[Multiple Result Sets]] for details on these changes:

* **QueryResults Methods and Results Classes** - were added so you can very easily parse multiple result sets without any additional code.

## Output Parameters ##

See [[Output Parameters]] for details on these changes:

* **QueryResults Returns OutputParameters** - if you use QueryResults instead of Query, you can access the output parameters through the Output property of the results.

## Dynamic Connections ##

See [[Dynamic Database Calls]] for details on these changes:

* **cancellationToken Parameter** - added to allow asynchronous methods to be cancelled
* **timeout Parameter** - The dynamic `timeout` parameter has been renamed to `commandTimeout` to better match the rest of the API.
* **returnType / withGraph / withGraphs Parameters** - The dynamic `returnType` parameter allows you to specify the type of object you want to return from the method without having to create a different instance of the `DynamicConnection`.

## Performance Optimizations ##

* **SequentialAccess Mode** - when Insight knows that it is going to parse a query (such as with Query, Insert, or other methods), it runs the query with CommandBehavior.SequentialAccess. The deserialization logic was rewritten to guarantee that it always reads the data in sequential order. Since we never rewind the data, this allows .NET to not cache row data, speeding up query reading and reducing memory footprint.
* **Optimized QueryIdentity and SchemaIdentity** - much of the runtime cost of Insight is matching a query to a parameter method or matching a recordset to a deserializer. This code was tuned using RedGate's Performance Profiler to optimize this path.
* **Optimized Object Graph Deserializer** - through some good refactoring, deserializing object graphs is now a little faster.
* **Async Uses ConfigureAwait** - async mode uses ConfigureAwait to have better performing continuations.
* **Removed SchemaTable Requirement** - most queries can be mapped without having to access the expensive SchemaTable.

## Other Stuff ##

* **Single** - extension methods added to return a single object from a result set.
