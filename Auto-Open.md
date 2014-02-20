Most of the Insight extension methods support auto-open/auto-close semantics. If you pass it an open connection, Insight will leave the connection open. If you pass it a closed connection, Insight will open the connection itself, and close the connection when it completes.

This call will automatically open and close the connection (note the lack of `using` statement):

	var beer = Database.Connection().Query<Beer>("FindBeer", new { Name = "IPA" });

This call will leave the connection open between statements:

	using (var connection = Database.Open())
	{
		var beer = connection.Query<Beer>("FindBeer", new { Name = "IPA" });

		// connection remains open

		connection.Execute("DeleteBeer", beer.First());
	}

Auto-open/auto-close makes it easy to use the Insight extension methods within existing code, or to optimize your connections by keeping it open for multiple queries. Most likely, the only time you need to open a connection is when you need to use a transaction.

All of the Insight methods that execute a query support this method. However, methods that let you access the data reader directly do not support Auto-Open/Close.

The methods that **do not** support auto-open are:

* CreateCommand
* GetReader

[[Opening Connections]] <BACK || NEXT> [[Executing SQL Commands]]
