Cloud database providers such as SQL Azure introduce some additional complexity for database connections. SQL Azure connections can have temporary errors that can cause connections to fail. The SQL Azure team has recommended that retry operations be supported in order to handle transient connections.

See [Best Practices for Handling Transient Conditions](http://windowsazurecat.com/2010/10/best-practices-for-handling-transient-conditions-in-sql-azure-client-applications/) for more information.

Insight makes this process much simpler by introducing the ReliableConnection class. It automatically handles retries for transient connection errors, in both synchronous and asynchronous scenarios.

## Basic ReliableConnection ##

Just replace your `SqlConnection` with `ReliableConnection<SqlConnection>`.

	using Insight.Database.Reliable;

	var connection = new ReliableConnection<SqlConnection>(connectionString);
	var results = connection.Query<Beer>("FindBeer", new { Name = "IPA" });

## Tweaking the RetryStrategy ##

You also have access to the parameters of the RetryStrategy.

	RetryStrategy.Default.MaxRetryCount = 5;
	RetryStrategy.Default.MinBackOff = new TimeSpan (0, 0, 0, 0, 100); // 100 ms
	RetryStrategy.Default.MaxBackOff = new TimeSpan (0, 0, 0, 1, 0); // 1 s
	RetryStrategy.Default.IncrementalBackOff = new TimeSpan (0, 0, 0, 0, 100); // 100 ms

## Retrying Event ##

You can also hook the Retrying event on a per-strategy basis:

	RetryStrategy.Retrying += (sender, re) => { Console.WriteLine("Retrying. Attempt {0}", re.Attempt); };

## Advanced Strategies ##

You can create multiple RetryStrategies and pass them into your ReliableConnection if you need different ones.

	var strategy = new RetryStrategy();
	var connection = new ReliableConnection<SqlConnection>(connectionString, strategy);

For more advanced retry strategies, you could implement your own IRetryStrategy, but it's pretty complicated. 

## Integration with the Microsoft Enterprise Library Transient Fault Handling Application Block ##

The Microsoft Enterprise Library team has a good Application Block that has a lot of options and works across SQL Azure, Azure Service Bus, Azure Storage and Azure Caching. This gives you consistent retry logic across the different types of resources.

You can integrate the Application Block by adding an adapter between IRetryStrategy and the Application Block RetryManager class.