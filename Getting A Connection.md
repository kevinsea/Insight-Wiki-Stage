Insight is a set of extension methods that make working with database connections a little easier. Most of them require an `IDbConnection`, so first you need to get one of them.

You can just create your connection the way you always have:

	var connection= new SqlConnection(connectionString);

	// run a query right off the connection (this performs an auto-open/close)
	connection.QuerySql("SELECT * FROM Beer");

Insight also provides extensions for getting connections from your configuration objects.

## ConnectionStringSettings.Connection ##

Converts a ConnectionStringSettings object to a SqlConnection.

	ConnectionStringSettings database = ConfigurationManager.ConnectionStrings["MyDatabase"];

	// run a query right off the connection (this performs an auto-open/close)
	database.Connection().QuerySql("SELECT * FROM Beer");

## DbConnectionStringBuilder.Connection ##

Converts a DbConnectionStringBuilder to a DbConnection. The type of connection is detected from the type of the builder.

	SqlConnectionStringBuilder database = new SqlConnectionStringBuilder(connectionString);
	// make other changes here

	// run a query right off the connection (this performs an auto-open/close)
	database.Connection().QuerySql("SELECT * FROM Beer");

NEXT: [[Opening Connections]]
