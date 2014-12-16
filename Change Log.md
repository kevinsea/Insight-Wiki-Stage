# Change Log #

## v5.1.1 ##

* Fixed Issue #168 - support for detecting ID fields ending in _id
* Fixed Issue #167 - dynamic parameters can now contain enumerables for TVPs
* Implemented Issue #164 - child recordsets can now map to single records
* Fixed Issue #171 - wrapped providerr did not unwrap bulkcopyasync calls
* Added missing RegisterProvider method for Postgres
* Fixed Issue #174 - sql parameters should be case-insensitive in sql text
* Implemented Issue #173 - new ExecuteXml methods for SqlConnections
* Implemented Issue #175 - added an automatic schema-changing wrapper for postgres

## v5.1 ##

* Support for non-default constructors
* Fixed #162/163 - bulk copy when a middle column is computed or readonly

## v5.0.3 ##

* Fixed Issue #156 - Derived classes could not override column name.
* Added WithoutNulls methods to FastExpando (see Issue #157).
* Fixed Issue #158 - BulkCopy fails when connection is wrapped with Glimpse.
* Fixed Issue #159 - SingleReader fails when no records are returned. 

## v5.0.2 ##

* Fixed Issue #154 - `ExecuteScalar<string>` fails when no records are returned.
* Added ArgumentNull checks for As extension methods.

## v5.0.1 ##

* Fixed Issue #152 - string output variables were sending the incorrect length to the server.

## v5.0.0 ##

* Allow parameters & values to map into subobjects.
* Interface methods automatically can map values into fields of objects passed in. See Issue #125.
* Updated configuration system for custom serialization and mapping.
* Support for composite keys in parent/child relationships. See issue #147.
* Support for parent IDs to be part of child objects instead of just outside of them. See issue #145.
* Optimized code generation by combining assemblies for static fields.

## v4.2.10 ##

* Fixed Issue #148 - generated interfaces are now in a single dynamic assembly.

## v4.2.9 ##

* Fixed Issue #142 - child record is atomic/value type.
* Fixed Issue #146 - SingleChildMapper throws when no parent record is returned.

## v4.2.8 ##

* Optimized IL for parameter generation.
* Implemented Issue #141 - easier custom record readers.

## v4.2.7 ##

* Fixed json package for NET35.
* Updated DynamicConnection to allow providers to fixup commands.
* Added friendlier error message when attempting to implement a private interface.
* Fixed Issue #136 - exception thrown when interface returns List and has child records.
* Added better exceptions when id/list fields not found on class.
* Implemented Issued #137 - BulkCopyAsync
* Fixed Issue #139 - support nullable return values in ExecuteScalar.

## v4.2.6 ##

* Fixed Issue #133 - allow custom deserialiers to deserialize any type, including strings
* Fixed Issue #132 - added more oracle transient error codes
* Made wrapped provider pass more calls to inner provider
* Added option to disable MergeOutput for interface methods

## v4.2.5 ##

* Fixed Issue #126 - Update methods now automatically merge outputs.
* Fixed Issue #129 - null subobjects didn't work due to sequential access on data reader
* Fixed Issue #130 - virtual id/list accessors for child relationships throws VerificationException

## v4.2.4 ##

* Fixed issue #123 - OpenWithTransaction now closes inner connection rather than disposing it, so the connection can be reused.
* Implemented Issue #124 - Oracle providers can now auto-detect output refcursors in SQLText, so you can now easily return multiple result sets.
* Removed call to System.Activator in cloning of wrapped connections

## v4.2.3 ##

* Fixed Issue #117 - Oracle providers now set BindByName on commands
* Fixed Issue #118 - DB2 and Oracle providers are now available for .NET 4.0

## v4.2.2 ##

* Updated library dependencies to latest versions.
* Fixed Issue #114 - wrapped connections throw during cloning in AsParallel implementations.
* Improved error message for empty result sets in calls to ExecuteScalar that return non-nullable values.

## v4.2.1 ##

* Fixed nuget package dependencies so providers depend on the latest version of the core library.
* Column overrides in OneToOne now work in child records.
* Private properties on parent classes can now be set.

## v4.2.0 ##

* Fixed Issue #111 - Xml Output Parameters
* Fixed Issue #109 - field objects can now be created with string constructors and converters
* Implemented Issue #112 - Added MergeOutput Attribute for interface methods
* Implemented Issue #109 - Added MultiReader class that allows different classes to be returned for each record in a stream
* Implemented Issue #110 - added CachedDbDataReader and PostProcessRecordReader


## v4.1.5 ##

* Fixed Issue #107 - VerificationException when accessing readonly field.
* Implemented Issue #106 - more methods for As and AsParallel.
* Fixes for TVPs in SQLText.

## v4.1.4 ##

* Missing provider exceptions are now more informative.
* `RegisterProvider` forces registration again so Insight can be used in add-in and other scenarios.
* Fixed schema identity caching issue when a table UDT matches a bulk copy table and their readonly properties do not match. 

## v4.1.3 ##

* Implmented Issue #101 - SingleReader.ThenChildren now ignores ID field and maps all children to parent (unless ID field or accessor is specified).
* Exposed RegisterProvider on all provider types so assembly references can be forced. See Issue #95.
* Fixed bulk copy operations for wrapped connections, and KeepIdentity options.
* Fixed Issue #102 - updated nuget packages and projects to not add a dependency on Microsoft.CSharp when building for NET35.
* Fixed issue where a read-only or identity field would make subsequent column mappings invalid.
* Abstract class implementation can now call a protected base class constructor.

## v4.1.2 ##

* Fixed provider auto-registration in more scenarios.
* Fixed issue where column mapping was dependent upon the order of members in the class.
* Query identities now differentiate between provider types.
* Fixed Issue #98 - computed columns in BulkCopy.
* Fixed Issue #97 - NullReference when calling ForEach with no results structure. 
* Can now auto-implement abstract classes derived from DbConnectionWrapper.
* Auto-implement now fills in a IDbConnection GetConnection() method if one exists on the class.

## v4.1.1 ##

* Updated provider packages to only depend on core library.
* Fixed issue where auto-registration did not work while running under IIS.
* ISSUE #94 - FastExpandos no longer convert properties to uppercase. They now do a case-insensitive key lookup.
* Better type support for dynamic parameters (e.g. using GUIDs with dynamic and SQL text.
* Fixed auto-implementing derived interfaces
* Auto-implementation now can auto-implement abstract classes.
* Optimized execution of AsParallel interfaces.


## v4.1.0 ##

* The built-in providers have been split out into Insight.Database.Providers.Default.
* New NuGet package Insight.Database.Core that omits the default providers. Handy for running on a platform that doesn't support the default providers.
* Insight will now auto-register any provider that is installed in the application folder. **Note: there is an issue with auto-registration under IIS, so using v4.1.0 is not recommended. This is fixed in v4.1.1**

## v4.0.1 ##

* ISSUE #87 - optimization - when sending an empty list to a TVP, omit the parameter value.

## v4.0.0 ##

* Added support for returning arbitrary object structures.

## v3.3.1 ##

* FIXED ISSUE #81 - inheriting schema attribute.

## v3.3.0 ##

* ISSUE #76 - support output parameters with async methods.
* FIXED ISSUE #77 - auto-interface can now return `IEnumerable<T>`, IList<T>, `or List<T>`.
* ISSUE #75 - allow dynamic calls to specify a database schema.
* Added [[Optimistic Concurrency]] support.
* Added connection.AsParallel<T> to allow [[Auto Interface Implementation]] to generate thread-safe interfaces. 

## v3.2.0 ##

* ISSUE #74 - strong-named assemblies. See [[Strong Named Assemblies]].

## v3.1.6 ##

* Fixed ISSUE #73 - exception when returning too few result sets with QueryResultsAsync
* Fixed ISSUE #72 - default datetime is now DbType.DateTime, unless the provider supports DateTime2 (i.e. SQL)

## v3.1.5 ##

* Added SqlAttribute.Schema to let you specify the schema for an entire set of procedures. (Issue #65)
* Updated SqlAttribute.CommandType to assume StoredProcedure if the sql is a single word or Text otherwise.
* Fixed Issue #67 - DateTime2 is now downshifted to DateTime when using SQL2005 or SQL CE.
* Fixed Issue #69 - double dispose on DbDataReader causes some third-party libraries to fail.
* Added basic SQLite tests.

## v3.1.4 ##

* Added automatic JSON Serialization (Issue #60). See [[Object Serialization]].
* Postgres provider now supports .NET v3.5 (Issue #58). Note: if you want other providers to support .NET 3.5, please post a github issue. It shouldn't be that much work for most of them.
* Fixed Issue #61 - when deserializing a sub-object, check to see if all of the sub-object's columns are dbnull. If so, return a null sub-object.
* Added project icon
* Fixed issue where commandTimeout was not being passed to a dynamic proc call

## v3.1.3 ##

* BulkCopy extension methods now return number of records inserted. (Issue #55, thanks eric-b)
* Enabled streaming for SqlBulkCopy provider.
* Added a BulkCopy extension that takes an IDataReader.

## v3.1.2 ##

* Added [[Sybase ASE Provider]] (Issue #52)
* Added XML Help files to nuget packages (Issue #51)
* Fixed Issue #50 - DynamicConnection may bypass Reliable features
* Fixed Issue #49 - When using ReliableConnection + Dynamic + Async OR OpenWithTransaction, Connections May Not Dispose Properly

## v3.1.1 ##

* Fixed build process to actually package NET35 and NET40 assemblies (whoops)
* Another Auto-Interface fix for string parameters and generics.

## v3.1 ##

* Added support for .NET 3.5
* Fixed Issue #48 - Auto-Interface Update Method Fails if First Parameter is atomic
	*  NOTE: Auto-Interface Update methods no longer perform a merge on the results. Use Upsert if that is the desired behavior. See [documentation](https://github.com/jonwagner/Insight.Database/wiki/Auto-Interface-Implementation)
* Fixed Issue #47 - bulk copy inside a transaction

## v3.0.4 ##

* Fixed issue with sending SqlGeometry to SQL Server calls. (Issue #43)
* Also optimized some parameter cloning & made SqlServer UDTs work better with SQL Text.

## v3.0.3 ##

* Added QueryResults overload with no type parameter for queries that return just output parameters.
* Fixed Issue #41 - non-sql exceptions causing provider lookup to fail in ReliableConnection
* Fixed Issue #42 - string to guid conversion in stored proc parameters

## v3.0.2 ##

* Added [[Glimpse Provider]] for use with the Glimpse testing framework.

## v3.0.1 ##

* Fixed issue where a missing execute permission on a UDT used as a parameter type was causing Insight to thro the wrong exception.
* Added [[DB2 Provider]].

## v3.0.0 ##

* Implemented a new provider model to support the following databases:
	* [[SQL Server Provider]]
	* [[Oracle Provider]]
	* [[ODBC Provider]]
	* [[OLEDB Provider]]
	* [[MiniProfiler Provider]]
* Some parameters for [[Bulk Copy with Objects]] have changed to better support multiple providers.
* Added QueryOnto extension methods. (See [[Identity Insersts]]).
* NOTE: if you are using MiniProfiler, you will need to register the MiniProfiler provider starting with v3.0
* Implemented Issue #33 - Updated some extension methods to use T4 templates and support more generics (Phil Bolduc)

## v2.3.3 ##

* Fixed Issue #36 - automatic conversion of guid to string in list parameters.

## v2.3.2 ##

* Fixed Issue #32 - now uses DbType.DateTime2 as the default datetime type for wider range of values.

## v2.3.1 ##

* Fixed Issue #31 - support for table types in schemas with dotted names.
* Fixed Issue #30 - support for case-sensitive databases.
* Updated support for all SQL Server User-Defined Types. (Re: Issue #26)

## v2.3.0 ##

* New outputParameter option for connection extension methods. See [[Output Parameters]].
	* **This causes the extension methods to be incompatible with versions before v2.3. A recompile takes care of this.**
* Implemented Issue #25 - support for output parameters on [[Auto-Interface Implementation]].
* Added support for SqlGeometry type. (Issue #26)

## v2.2.2 ##

* Implemented Issue #23 - additional options for configuring SqlBulkCopy.

## v2.2.1 ##

* Fixed duplicate XML encoding when converting a string field to an xml column in a table-valued parameter.

## v2.2.0 ##

* **Fixed Issue #22 - Missing Table Parameters to a stored procedure now throws an InvalidOperationException.** 
* If you are calling stored procedures and omitting table parameters, relying on the default behavior of SQL server to provide an empty table, please review your code before installing this version. Insight will now throw an InvalidOperationException, to warn you that you are missing a table parameter. (Without this, it's easy to mask a bunch of bugs.) Instead you can pass in an empty list or the new Parameters.EmptyList property:
	* connection.Query("MyProc", new { Table = Parameters.EmptyList }	
	* connection.MyDynamicProc(table: Parameters.EmptyList)

## v2.1.5 ##

* Fixed issue #21 - ObjectReader can now read byte[] and send to binary[]

## v2.1.4 ##

* Better automatic conversions from T => string. e.g. Guid => string.
* Removed dependency on FxCop for code analysis. Now using Visual Studio Code Analysis.
* Added more reliability around exception handling and disposing connections.
* Updated build process to have fewer required dependencies.

## v2.1.3 ##

* Generic methods now accept `<dynamic>`, such as `QueryResults<dynamic, dynamic>` or `ToList<dynamic>`. One thing to note: `Query<object>` will now give you a FastExpando, but it's still an object...
* Better error message if an exception occurs during deserialization type conversion.

## v2.1.2 ##

* Better type conversions for reading values from objects for list parameters and bulkcopy.
* Better type conversion between .NET TimeSpan and SQL time. See [[Date-Based Data Types]].

## v2.1.1 ##

* **Insight now supports [[Auto Interface Implementation]]. I've been wanting this for 8 years!**
* Opening connections through DbConnectionStringBuilder or ConnectionStringSettings now return the proper connection type (not just SqlConnection).
* **Extension methods for ConnectionStringSettings now return a DbConnection rather than a SqlConnection.** You may get compiler errors if you use these methods. You can cast the return type to SqlConnection or use var as appropriate.
* For ease of use with the above change, the BulkCopy extension method now accepts any type of DbConnection, but will throw if it's not a SqlConnection.
* Added extension methods for OpenAsync, OpenWithTransaction, and OpenWithTransactionAsync. **See [[Transactions]] for great new syntax.**
* Added extension methods for Single and SingleAsync.
* Added support for deserializing structs.

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