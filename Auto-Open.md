# Auto-Open / Auto-Close #

Most of the Insight extension methods support auto-open/auto-close semantics. If you pass it an open connection, Insight will leave the connection open. If you pass it a closed connection, Insight will open the connection itself, and close the connection when it completes.

This call will open and close the connection:

			List<Beer> beer = Database.Connection().Query<Beer>("FindBeer", new { Name = "IPA" });

This call will leave the connection open between statements:

			using (SqlConnection connection = Database.Open())
			{
				List<Beer> beer = connection().Query<Beer>("FindBeer", new { Name = "IPA" });
				connection().Execute("DeleteBeer", beer.First());
			}

This makes it easy to use the Insight extension methods within existing code.

The methods that support this are:

* ExecuteAsync/ExecuteSqlAsync
* QueryAsync/QuerySqlAsync
* BulkCopy
* Execute/ExecuteSql
* ExecuteScalar/ExecuteScalarSql
* ForEach
* Query/QuerySql