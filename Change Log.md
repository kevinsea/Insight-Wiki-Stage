# Change Log #

## v2.1 (coming soon) ##

* **Insight now supports [[Auto Interface Implementation]]. I've been wanting this for 8 years!**
* Opening connections through DbConnectionStringBuilder or ConnectionStringSettings now return the proper connection type (not just SqlConnection).
* **Extension methods for ConnectionStringSettings now return a DbConnection rather than a SqlConnection.** You may get compiler errors if you use these methods. You can cast the return type to SqlConnection or use var as appropriate.
* For ease of use with the above change, the BulkCopy extension method now accepts any type of DbConnection, but will throw if it's not a SqlConnection.
* Added extension methods for OpenAsync, OpenWithTransaction, and OpenWithTransactionAsync. **See [[Transactions]] for great new syntax.**
* Added extension methods for Single and SingleAsync.

## v2.0.2 ##

* Fixed issue #13 - dynamic calls no longer hide SqlExceptions.
* Fixed issue #14 - nullable parameters were not mapped properly.
* Fixed issue #15 - single-column result sets did not deserialize nullables properly.

## v2.0.1 ##

* v2.0 is almost a total rewrite of the library, and it's everything you wanted for the holidays. See [[What is New in v2.0]].

## v1.2.14 (Unreleased) ##

* Added ExecuteScalarAsync.
* Fixed Issue #11 - not having EXECUTE access to a user-defined type caused the parameter to go missing with no error.

## v1.2.13 ##

* Fixed issue #10 - autoopen with procs that take lists of objects.

## v1.2.12 ##

* Added OutputParameters<T>() to return a new object constructed from output parameters.

## v1.2.11 ##

* Made sure code snippets are included in the NuGet package.
* Added support for [[Output Parameters]].

## v1.2.10 #

* Fixed support for time columns and TimeSpan values.

## v1.2.9 ##

* Fixed Execute stored proc with Table-Valued Parameter through a ReliableConnection.

## v1.2.8 ##

* Added ForEach and MultiResults code snippets. Reinstall your snippets to get them.
* FIX: ExecuteAsync was failing on DeriveParameters with a Closed connection.

## v1.2.7 ##

* Much better support when coercing types between SQL and CLR. Now can convert decimal -> int, int64 -> enum32, etc.
* Fixed issued #5 related to SCOPE_IDENTITY() returning numeric (i.e. decimal) and needing to coerce it to an int CLR field.

## v1.2.6 ##

* Support for converting different underlying types between Enums and DB. Now you can use tinyint in the database, and long for your Enums.

For breaking changes: **bold** changes require code changes. *Italic* changes just require recompilation.

## v1.2.5. ##

* Added even more methods to the repository [[Code Snippets]] to be compatible with Insight.Database.Schema AutoProcs.
* Updated sample code to match latest Insight.Database.Schema AutoProcs.

## v1.2.4 ##

* ReliableConnection and ReliableCommand are now Derived from DbConnection and DbCommand for better support of async in .NET 4.5.
* Added more exception handling for RetryStrategy.ExecuteWithRetryAsync.
* Added Visual Studio [[Code Snippets]] for repository generation.
* Field/Property binding is not case-insensitive.

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