# Change Log #

For breaking changes: **bold** changes require code changes. *Italic* changes just require recompilation.

## v1.2.3 ##

* Added some explicit `using`s around IDataReader for code explicityness.
* Fixed returning single column of byte[].

## v1.2.2 ##

* NuGet package now ships .NET 4.0 and .NET 4.5 assemblies.
* .NET 4.5 assembly now does fully asynchronous opens. No code changes on your part.
* .NET 4.5 assembly automatically uses .NET 4.5 async SQL tasks. No code changes on your part.

See the bottom of [[Async Commands and Queries]] for some async notes.

## v1.2.1 ##

* Added `ReliableConnection`, which automatically retries database queries when a transient exception is encountered.
* *Added Async support for ReliableConnection, which required that the Async extensions be switched from SqlConnection to IDbConnection. This is better anyway, since additional connection types will eventually support async commands.*

## v1.1.5 ##

* Added compatibility with [MiniProfiler](http://miniprofiler.com/), so that when you wrap your SqlConnection in a ProfiledDbConnection, we can still detect stored procedure parameters.
* Added support for `IDataReader.Merge`, that lets you merge results onto an existing object or list of objects. Handy for Identity Insert.
* Added `Insert(Sql)/InsertList(Sql)` methods that let you execute an Insert statement, then take returned identities and merge them into the inserted objects. Equivalent to `IDBConnection.GetReader("INSERTPROC", object(s)).Merge(object(s))`
* Added Async equivalent methods for Insert/InsertList.
* Calling a stored proc with one table type parameter, and passing in an IEnumerable<T> should map properly. Handy for inserts.

## v1.1.4 ##

* Added caching to `ObjectListDbReader` to improve performance and eliminate an annoying exception that pops up while debugging.

## v1.1.2 ##

* Passing a single object to [[Dynamic Database Calls]] now results in the object's properties being mapped to parameters, just like with `Execute` and `Query` methods.

## v1.1.1 ##
Now in NuGet!

v1.1 contains a few breaking changes. 

* Added support for ColumnAttribute-based mapping.
* Added support for [[Dynamic Database Calls]] with a handy `connection.ProcName(params)` syntax.
* *FastExpando.FromObject and .Expand no longer take type arguments and now pull properties out of the object's runtime type, not compiled type*. 
* *Insight methods now return IList<T> instead of List<T> for future-proofing*. It's better for the environment, at the cost of losing .ForEach and .Sort. You'll cope. Or yell at Microsoft.
* **Changed AsyncXXX methods to XXXAsync for consistency**.

## v1.0.55 ##
* Fixed multi-class deserializer in jit-optimized scenarios.