# Async Commands and Queries #

If you want to write a scalable and responsive application, you need to stop blocking your threads. Asynchronous programming lets you write code that waits for nobody. Fortunately, with the Task Parallel Library in the .NET Framework, this is now a **lot** easier. Unfortunately, async programming with the SQL library isn't so easy yet. (NOTE: Insight is coded against .NET 4.0, so the .NET 4.5 ADO.NET library is probably much better.)

Insight allows you to call the following methods in asynchronous mode:

* ExecuteAsync / ExecuteSqlAsync
* QueryAsync / QuerySqlAsync
* ToList

## Before You Begin ##
First, make sure you are using a data provider that supports asynchronous calls. Then, make sure that your connection string contains:

	AsynchronousProcessing=true

## Async Execute ##
You can execute a command and get a task back that represents the completion of the call:

	Task task = Database.Connection().ExecuteAsync("InsertBeer", beer);

	// do stuff

	task.Wait();

This also works with ExecuteSqlAsync.

## Important Notes ##

* The Insight library handles the connection lifetime with the auto-open/auto-close semantics.
* Make sure you do not pass the same database connection to multiple AsyncXXX calls. You have no guarantee about the order and they may attempt to open and close the connections at the same time. (Hint: use ConnectionSettings or SqlConnectionStringBuilder as your starting class and create the connections as above.)

## QueryAsync ##
Querying for objects works the same way, except the result of the task is a List<T> or List<FastExpando>.

	Task<IList<Beer>> task = Database.Connection().QueryAsync<Beer>("FindBeer", new { Name = "IPA" });

	// do stuff

	var results = task.Result;

	foreach (Beer b in results)
		Console.WriteLine(b.Name);

## ToList ##
If you have a Task<IDataReader>, you can also call the extension method ToList<T> to convert the recordset to a list of objects upon the completion of the task.

	Task<IDataReader> task = null;	// got this somehow
	Task<IList<Beer>> beerTask = task.ToList<Beer>();

This lets you use the object mapping engine, but manage the rest of the connection and query lifecycle yourself.

## Some Async Notes ##

* .NET 4.5 now supports OpenAsync on SqlConnections. Insight will use this in async mode when it is available. If OpenAsync is not available on the given type of connection, it falls back to a task that blocks on the open call.
* Insight does not use Async mode for deriving the stored procedure parameters. Parameter derivation only should occur once per query, so it's a blocking operation. We didn't want to add a whole bunch of task infrastructure for that one-time startup scenario.