# Using Transactions and Insight.Database #

Sometimes you want to execute multiple statements in a transaction. We recommend using the transaction support in the DbConnection classes. You will need to manage the lifetime of your connection explicitly.

You also need to pass the transaction into each database call.

	using (var c = connectionString.Open())
	using (var tx = connection.BeginTransaction())
	{
		c.Execute("InsertBeer", beer, transaction: tx);
		c.Execute("InsertBeer", beer, transaction: tx);
		c.Commit();
	}

## Using OpenWithTransaction ##

I hate code, so when I saw this, I set out to eliminate the code. Calling OpenWithTransaction will open the connection AND start a transaction. It will also automatically pass the transaction to all of your database calls.

	using (var c = connectionString.OpenWithTransaction())
	{
		c.Execute("InsertBeer", beer);
		c.Execute("InsertBeer", beer);
		c.Commit();
	}

## Using OpenWithTransaction and Async ##

You can also do this in an async method.

	public async Task InsertWithTransaction()
	{
		using (var c = await connectionString.OpenWithTransactionAsync())
		{
			await c.ExecuteAsync("InsertBeer", beer);
			await c.ExecuteAsync("InsertBeer", beer);
			c.Commit();
		}
	}

Notes: 

1. Don't even think of trying to write this without .NET 4.5. Cleaning up the connection would be a nightmare.
2. Note that we await the result of each database call so they are executed in order, but asynchronously. I haven't tried executing them in parallel with MARS enabled.
