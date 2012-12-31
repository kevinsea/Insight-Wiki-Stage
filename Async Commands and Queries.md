# Async Commands and Queries #

If you want to write a scalable and responsive application, you need to stop blocking your threads. Asynchronous programming lets you write code that waits for nobody. Fortunately, with the Task Parallel Library in the .NET Framework, this is now a **lot** easier. Unfortunately, async programming with the SQL library isn't so easy yet. Insight wraps all of the async logic for you.

Insight allows you to call the following methods in asynchronous mode:

* ExecuteAsync / ExecuteSqlAsync
* ExecuteScalarAsync / ExecuteScalarSqlAsync
* InsertAsync / InsertSqlAsync
* InsertListAsync / InsertListSqlAsync
* MergeAsync
* QueryAsync / QuerySqlAsync
* QueryResultsAsync / QueryResultsSqlAsync
* ToListAsync

## Before You Begin ##
First, make sure you are using a data provider that supports asynchronous calls. Then, make sure that your connection string contains:

	Asynchronous Processing=true

NOTE: In .NET 4.0, only the SQL Server Provider supports async. In .NET 4.5, most of the built-in providers have an async mode, but only the SQL Server Provider currently uses async all the way through the API.

## Async Extension Methods ##
Most of the Insight extension methods have an asynchronous version that ends in `Async`. These methods are the same as the synchronous methods, but they return a `Task<T>` rather than the basic return type.

You can execute a command and get a task back that represents the completion of the call:

	public Task<int> InsertBeer(Beer beer)
	{
		return Database.Connection().ExecuteAsync("InsertBeer", beer);
	}

This also works with ExecuteSqlAsync.

In .NET 4.5, you can use this with the async/await pattern for easy full-async methods. This method inserts a new beer and returns its ID.

	public async Task<int> InsertBeer(Beer beer)
	{
		var results = await Database.Connection().InsertAsync("InsertBeer", beer);

		return results.First().ID;
	}

## Performance Notes ##

In .NET 4.5, Insight performs record reads (`ReadAsync`) and recordset advances (`MoveNextAsync`) asynchronously. This allows .NET to advance the stream on network packet without blocking the thread. Insight does not use field-level asynchronous methods, because you rarely need to use that level of granularity for asynchronous reads.

## Important Notes ##

* The Insight library handles the connection lifetime with the auto-open/auto-close semantics **even in async mode**. You don't have to worry about opening and closing connections or DataReaders.
* Make sure you do not pass the same database connection to multiple AsyncXXX calls. You have no guarantee about the order and they may attempt to open and close the connections at the same time. (Hint: use ConnectionSettings or SqlConnectionStringBuilder as your starting class and create the connections just like in the code above.)
* .NET 4.5 now supports OpenAsync on SqlConnections. Insight will use this in async mode when it is available. If OpenAsync is not available on the given type of connection, it falls back to a task that blocks on the open call.
* Insight does not use Async mode for deriving the stored procedure parameters. Parameter derivation only should occur once per query, so it's a blocking operation.