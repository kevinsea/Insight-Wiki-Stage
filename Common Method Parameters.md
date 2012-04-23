# Common Method Parameters #

## parameters ##
This is the object to use to set the parameters on the query. See [[Query Parameter Mapping]] for more information on how the parameters are mapped.

## commandType ##
Setting the commandType parameter allows you to change the CommandType that is being executed. If this is not specified, then CommandType.StoredProcedure is assumed. You won't use this often, because most methods that accept commandType also have an XXXSql method.

	// notice that we use Query here, but change the CommandType
	var beer = Database.Query("SELECT * FROM Beer", Parameters.Empty, commandType: CommandType.Text);

## commandTimeout ##
Setting commandTimeout changes the number of seconds that the command will wait before timing out.

	// this could take an hour to brew...
	var beer = Database.Query("BrewBeer", Parameters.Empty, commandTimeout: 60 * 60);

## transaction ##
If you are executing commands within an IDbTransaction, pass in the transaction so that the commands can participate the transaction. If you are using this, then you probably need to manage the connection's lifecycle manually.

	static void Parameter_Transaction()
	{
		using (SqlConnection connection = Database.Open())
		using (SqlTransaction t = connection.BeginTransaction())
		{
			Beer beer = new Beer("Sly Fox IPA");
			connection.Execute("InsertBeer", beer, transaction: t);

			// without a commit this rolls back
		}
	}

## read ##
This defines a function that allows you to manually transform an IDataReader result to an object. If you have specialized deserialization needs, this can be helpful. See [[Manual Transformations]] for more information.

## action ##
This defines an action to apply to each record as it is retrieved from the database. If you are working with large result sets, it may be better to work on one record at a time. See [[Streaming Results Efficiently]] for more information.

## closeConnection ##
Setting this to true will tell the system to close the connection at the end of the command. This is mostly used internally for handling auto-open/close.

## commandBehavior ##
Setting the commandBehavior parameter allows you to change the behavior of commmand. You aren't likely to use this. In fact, I had to look up the flags in help, so they must not be that common.

