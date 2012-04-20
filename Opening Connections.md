# Opening Connections with Insight.Database #

These extension methods make working with connections a little more fluent.

We recommend that you use a DI tool to inject one of these types into your code, then use these extensions to write pretty code:

	class Refrigerator
	{
		// use your favorite DI tool here
		[Inject]
		private ConnectionStringSettings Database;

		public List<Beer> GetBeer(string name)
		{
			return Database.Connection().Query<Beer>("FindBeer", new { Name = "name" });
		}
	}

Some notes:

1. You probably will use the *Connection* methods much more than the *Open* methods, since Insight supports auto-open/close and does all of the hard work for you.
1. If you are doing asynchronous programming, it's better to use ConnectionStringSettings or SqlConnectionStringBuilder, so you don't accidentally send more than one query through one connection at a time.

## IDBConnection.OpenConnection ##
Opens the connection and returns it so you can chain your methods.
This is handy in a `using` statement when you want to create a connection and open it.

	using (SqlConnection c = new SqlConnection(connectionString).OpenConnection())
	{
		// do stuff...
		c.QuerySql("SELECT * FROM Beer", Parameters.Empty);
	}

Or start using the connection immediately...

	SqlConnection c = new SqlConnection(connectionString);
	c.OpenConnection().ExecuteSql("DELETE FROM Beer", Parameters.Empty); // nooooooo....

## ConnectionStringSettings.Connection ##
Converts a ConnectionStringSettings object to a SqlConnection. *NOTE: This currently assumes that you want a SqlConnection.*

	ConnectionStringSettings database = ConfigurationManager.ConnectionStrings["MyDatabase"];

	// run a query right off the connection (this performs an auto-open/close)
	database.Connection().QuerySql("SELECT * FROM Beer", Parameters.Empty);

## ConnectionStringSettings.Open ##
Converts a ConnectionStringSettings object to a SqlConnection and opens it. *NOTE: This currently assumes that you want a SqlConnection.*

	ConnectionStringSettings database = ConfigurationManager.ConnectionStrings["MyDatabase"];

	// get an open connection from the ConnectionStringSettings
	using (SqlConnection c = database.Open())
	{
		c.QuerySql("SELECT * FROM Beer", Parameters.Empty);
	}

## SqlConnectionStringBuilder.Connection ##
Converts a SqlConnectionStringBuilder to a SqlConnection.

	SqlConnectionStringBuilder database = new SqlConnectionStringBuilder(connectionString);
	// make other changes here

	// run a query right off the connection (this performs an auto-open/close)
	database.Connection().QuerySql("SELECT * FROM Beer", Parameters.Empty);

## SqlConnectionStringBuilder.Open ##
Converts a SqlConnectionStringBuilder to a SqlConnection and opens it.

	SqlConnectionStringBuilder database = new SqlConnectionStringBuilder(connectionString);
	// make other changes here

	// manage the lifetime ourselves
	using (SqlConnection c = database.Open())
	{
		c.QuerySql("SELECT * FROM Beer", Parameters.Empty);
	}