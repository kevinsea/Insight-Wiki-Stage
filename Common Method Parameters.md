# Common Method Parameters #

The Insight extension methods all take a common set of method parameters, discussed below.

## action ##
This defines an action to apply to each record as it is retrieved from the database. If you are working with large result sets, it may be better to work on one record at a time. See [[Streaming Results Efficiently]] for more information.

## cancellationToken ##

Specifies a `CancellationToken` that can be used to cancel an asynchronous operation.

## closeConnection ##
Setting this to true will tell the system to close the connection at the end of the command. This is mostly used internally for handling auto-open/close.

## commandBehavior ##
Setting the `commandBehavior` parameter allows you to change the behavior of commmand. The two most common cases are `CommandBehavior.CloseConnection`, which closes the connection at the end of the command, and `CommandBehavior.SequentialAccess`, which tells .NET to optimize access to the data for reading the fields in order. All of the Insight deserialization methods support SequentialAccess mode.

## commandTimeout ##
Setting `commandTimeout` changes the number of seconds that the command will wait before timing out.

	// this could take an hour to brew...
	var beer = Database.Query("BrewBeer", Parameters.Empty, commandTimeout: 60 * 60);

## commandType ##
Setting the `commandType` parameter allows you to change the CommandType that is being executed. If this is not specified, then `CommandType.StoredProcedure` is assumed. You won't use this often, because most methods that accept `commandType` also have an `XXXSql` method.

	// notice that we use Query here, but change the CommandType
	var beer = Database.Query("SELECT * FROM Beer", Parameters.Empty, commandType: CommandType.Text);

## inserted ##
For Insert methods, this allows you to override the object that is being inserted. When Insight deserializes the results from an Insert method, it will write the results back to the object passed in as the `inserted` parameter. By default, Insight assumes that the object passed in as the `parameters` parameter.  See [[Identity Inserts]] for more information about Insert queries.

## parameters ##
This is the object to use to set the parameters on the query. See [[Query Parameter Mapping]] for more information on how the parameters are mapped.

## read ##
This defines a function that allows you to manually transform an IDataReader result to an object. If you have specialized deserialization needs, this can be helpful. See [[Manual Transformations]] for more information.

## returnType ##
For Dynamic Database Calls, this allows you to specify the type of object you want returned from the method. This is equivalent to the generic type parameter on `Query<T>` methods. See [[Dynamic Database Calls]] for more information about the dynamic method syntax.

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

## withGraph / withGraphs ##
For Query and QueryReader methods, allows you to specify the types of the subobjects to be returned from the query. For example:

	var results = connection.Query<Beer, Glass, Serving>(
		"SelectServing", 
		new { servingID = 1 });

Can also be written as:

	var results = connection.Query<Beer>(
		"SelectServing", 
		new { servingID = 1 }, 
		withGraph: typeof(Graph<Beer, Glass, Serving>));

This is necessary for specifying the graphs for multiple recordsets when calling QueryResults.

	var results = connection.QueryResults<Beer, Beer>(
		"SelectTwoRecordsetsOfBeer", 
		new { servingID = 1 }, 
		withGraphs: new [] { typeof(Graph<Beer, Glass, Serving>), typeof(Graph<Beer, Glass>) });

Note that with the new DefaultGraph Attribute, you may not need to ever manually specify the graph for your queries. See [[Object Graphs]] for more information about the DefaultGraph Attribute.