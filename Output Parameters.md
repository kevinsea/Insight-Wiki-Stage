Sometimes your stored procedure needs to output parameters or return values.

## Return Values in SQL ##

If your stored procedure returns a value, the return value is sent back as a special parameter called "RETURN_VALUE".

	CREATE PROCEDURE AddOne_WithReturnValue @p int
	AS
		RETURN @p + 1
	GO
	
	DECLARE @i [int]
	EXEC @i = AddOne_WithReturnValue 1
	PRINT @i

## Output Parameters in SQL ##

If your stored procedure has OUTPUT parameters, they will be returned by name. In this case, there is a parameter named "p" that will receive the output parameter.

	CREATE PROCEDURE AddOne_WithOutputParameter @p int OUTPUT
	AS
		SET @p = @p + 1
	GO
	
	DECLARE @i [int] = 1
	EXEC AddOne_WithOutputParameter @i OUTPUT
	PRINT @i

Important Note: In SQL Server, OUTPUT parameters are actually INPUT/OUTPUT parameters. Therefore, if you declare an OUTPUT parameter, you must also pass in the parameter when calling the stored procedure.

If you don't want to pass in the parameter as an input, you can declare the parameter with a default when defining your stored procedure:

	CREATE PROCEDURE AddOne_WithOutputParameter @p int = NULL OUTPUT

## Putting Output Parameters into a Different Object ##

By default, Insight will attempt to overlay output parameters back onto the input parameters object.
You can have Insight try to put the output parameters onto a different object by passing in `outputparameters`.

The simplest way to get back output parameters is to put them back in an output parameters object:

	class OutputParameters
	{
		int p;
	}

	var outputs = new OutputParameters();
	connection.Execute("MyProc", outputParameters: outputs);
	var p = outputs.P;

## Accessing Output Parameters with QueryResults ##

If you use QueryResults/QueryResultsSql to execute your query, the results object automatically contains an Outputs property that contains the output parameters. It's a dynamic, so you can just access the properties directly.

	var results = connection.QueryResults<T>("MyProc", inputParameters);
	var outputs = results.Outputs;
	var p = outputs.P;

If your procedure has no result recordset, you can still call QueryResults, note how we pass in Results for the generic type parameter:

	var results = connection.QueryResults<Results>("MyProc", inputParameters);
	var p = results.Outputs.p;

You can also call QueryResults with no type parameter. this is the same as above:

	var results = connection.QueryResults("MyProc", inputParameters);
	var p = results.Outputs.p;

## Accessing Stored Procedure Return Values ##

SQL Server automatically adds an output parameter called "Return_Value" for the return value of a procedure. You can also get that from the results object:

	var results = connection.QueryResults<Results>("MyProc", inputParameters);
	var p = results.Outputs.Return_Value;

## Accessing Output Parameters on an IDbCommand ##

You can also access the output parameters and return values through the command object.

	var command = connection.CreateCommand("AddOne_WithOutputParameter");
	var results = connection.Query<SomeObject>(command);
	
	// OutputParameters() - returns a dynamic object containing all of the output parameters
	dynamic output = command.OutputParameters();
	int p = output.p;

If you use this method, you can also put the output parameters back into any existing object that has set properties or fields.

	class Foo
	{
		public int P { get; private set; }
	}

	// OutputParameters (object o) - copies output parameters into the fields/properties of the given object
	Foo output = new Foo();
	command.OutputParameters(output);

Or you can create a new object filled in with the output parameters.

	Foo output = command.OutputParameters<Foo>();

There are three versions of the OutputParameters extension method:

* OutputParameters() - returns a dynamic object that you can use to access the parameters with a dot syntax.
* OutputParameters<T>(T o) - copies the output parameters into the fields/set properties on the given object. This performs the same types of conversions that ToList<T> does (string to XML, XML to object, etc.)
* OutputParameters<T>() - returns a new instance of type T with the output parameters filled in.

[[Optimistic Concurrency]] <BACK || NEXT> [[Query Parameter Mapping]]