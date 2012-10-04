# Output Parameters and Return Values #

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

## Accessing Output Parameters with Insight ##

As of v1.2.12, Insight now properly defines parameters when calling stored procedures with output parameters or return values. You can access the output parameters and return values through the command object.

	var command = connection.CreateCommand("AddOne_WithOutputParameter");
	var results = connection.Query<SomeObject>(command);
	
	int p = command.Parameters["@p"].Value;

For convenience, Insight now defines some extension methods to make it easy to work with the output parameters.

	// OutputParameters() - returns a dynamic object containing all of the output parameters
	dynamic output = command.OutputParameters();
	int p = output.p;

You can also put the output parameters back into any existing object that has set properties or fields.

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

## Design Considerations for Output Parameters ##

We were hoping to make this even easier to use, but there are a bunch of language issues that make it difficult to make it any easier. See [[Design Considerations for Output Parameters]].
