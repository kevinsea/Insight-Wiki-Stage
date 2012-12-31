# Auto-Open / Auto-Close #

Most of the Insight extension methods support auto-open/auto-close semantics. If you pass it an open connection, Insight will leave the connection open. If you pass it a closed connection, Insight will open the connection itself, and close the connection when it completes.

This call will open and close the connection:

			List<Beer> beer = Database.Connection().Query<Beer>("FindBeer", new { Name = "IPA" });

This call will leave the connection open between statements:

			using (SqlConnection connection = Database.Open())
			{
				List<Beer> beer = connection.Query<Beer>("FindBeer", new { Name = "IPA" });
				connection.Execute("DeleteBeer", beer.First());
			}

This makes it easy to use the Insight extension methods within existing code, or to optimize your connections by keeping it open for multiple queries.

All of the Insight methods that execute a query support this method. However, methods that let you access the data reader directly do not support Auto-Open/Close:

The methods that **do not** support this are:

* CreateCommand
* GetReader

