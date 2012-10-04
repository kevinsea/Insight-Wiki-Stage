# Design Considerations for Output Parameters #

One of the main goals for Insight is to make database code easy to write and read, and have it just work.

## Attempt #1 - Anonymous Parameter Classes - FAIL ##

The first idea we had for storing output parameters was to put the values back into the input parameters. The code would look something like this:

	command.Execute("MyProc", new { inputParam = "foo", outputParam = (int)null });

Unfortunately, you wouldn't be able to access the output parameter, because it's stuffed back into a temporary object in the argument list. So you would have to write it like this:

	var parameters = new { inputParam = "foo", outputParam = (int)null };
	command.Execute("MyProc", parameters);

Unfortunately, in the CLR (or at least C#), anonymous classes are for read-only properties. The class is actually emitted something like this:

	class Anonymous
	{
		public int outputParam { get { return <outputParam>i_Value; } };
		private readonly int <outputParam>i_Value = null;
	}

Since the properties of the anonymous class are read-only, there is no way for Insight to set the fields on the anonymous parameter object.

## Attempt #2 - Explicit Parameter Classes - FAIL ##

Obviously, in order to have a place to store the objects, we would need an explicit class. Then your code would look like this:

	class OutputParameters
	{
		public int outputParam;
	}

	public void MyMethod()
	{
		var parameters = new OutputParameters();
		command.Execute("MyProc", parameters);
	}

This code just got big, and you should probably just use the SqlCommand class directly, right?

Well, technically this would work, except that some of the DbConnection extension methods (e.g. Execute and Query) take parameter objects, and other versions of the same methods take a DbCommand. In the case of DbCommand, there would be no object to stuff the output parameters into. This inconsistency would make the API difficult to use.

## Attempt #3 - Add an Optional Output Parameter Parameter - FAIL ##

One thing we could do is to add an optional output parameter parameter to the extension methods. The extension methods already have a few optional parameters, so what's another one? Like this:

	var parameters = new OutputParameters();
	command.Execute("MyProc", parameters, outputInto: parameters);

This wouldn't be too bad from a coding perspective, but it would change the signatures of all of the extension methods. This would require all existing applications to recompile against the updated library. Unfortunately, Insight.Database.Schema now uses Insight.Database internally, so it could cause conflicts with applications using an earlier or later version of Insight.Database.

## Attempt #4 - FastExpando Parameters - FAIL ##

The FastExpando class is really handy for defining dynamic objects that have properties that change at runtime. It's perfect for accepting the output parameters without having to define a class.

	var parameters = new FastExpando();
	command.Execute("MyProc", parameters);

This is pretty clean code-wise, but there is still an inconsitency between extension methods that take DbCommand and object parameters. At this point, we also expected that it would be difficult to add this to the async methods.

## Attempt #5 - DbParameter Wrapper ##

We also tried to wrap the DbParameter with another class that would transfer the value of the output parameter to the object when the Value set property was called. That would have been the cleanest. However, SqlParameterCollection won't accept anything other than SqlParameter, and the SqlParameter class was sealed.

## Realization - Output Parameters ~~ Multiple Recordsets ##

One way of thinking about output parameters is that they are like returning multiple recordsets from the stored procedure. So it's not too hard to imagine breaking your query to return more than one output.

	var command = connection.CreateCommand("MyProc");
	var result = connection.Query<T>(command);
	var output = command.OutputParameters();

	Console.WriteLine(output.OutputParam);

This is already a common pattern for multiple recordsets(although that uses GetReader and you have to control the lifetime of the connection).

This method has a minimal impact on the current API, and makes code cleaner, shorter, and easier to read. It also doesn't prevent us from updating the API later with extension methods that wrap these operations together. You are also free to examine the command.Parameters collection and handle the parameters manually.

There are three versions of the OutputParameters extension method:

* OutputParameters() - returns a dynamic object that you can use to access the parameters with a dot syntax.
* OutputParameters<T>(T o) - copies the output parameters into the fields/set properties on the given object. This performs the same types of conversions that ToList<T> does (string to XML, XML to object, etc.)
* OutputParameters<T>() - returns a new instance of type T with the output parameters filled in.