# Dynamic Database Calls #

For even better syntactic sugar, try using the `.Dynamic` extension method for connections. It lets you treat stored procedures as methods on your connection:

	IList<Beer> beer = Database.Dynamic<Beer>().FindBeer("IPA");

The type parameter on Dynamic is used to determine the type of object that is deserialized from the results of the query. But rather than using the generic type parameter on Dynamic, you can also use the returnType and withGraph parameters to control the types of objects returned. (Now, in Insight v2.0, you can now reuse the DynamicConnection for multiple calls and different return types.)

	IList<Beer> beer = Database.Dynamic().FindBeer("IPA", returnType: typeof(Beer));

Or

	IList<Beer> beer = Database.Dynamic().FindBeer("IPA", withGraph: typeof(Graph<Beer, Glass, Serving>));

## Parameters ##

By default, the parameters are passed to the stored procedure in order, but you can also use named parameters:

	IList<Beer> beer = Database.Dynamic<Beer>().FindBeer(name: "IPA");

If you are operating within a transaction, you can also use the reserved `transaction` parameter:

	using (SqlConnection connection = Database.Open())
	using (SqlTransaction t = connection.BeginTransaction())
	{
		IList<Beer> beer = connection.Dynamic<Beer>().FindBeer(name: "IPA", transaction: t);
	}

Note that since this is a dynamic method, the return type is of type `dynamic`. If you do not cast the return type to a known type, C# will invoke further method calls as dynamic invocations.

	// beer in this case is [dynamic beer]
	var beer = Database.Dynamic<Beer>().FindBeer("IPA");

	// First is invoked as a dynamic method
	Beer firstBeer = beer.First();

There is a small overhead for this type of call, but it's not much. The parameter definitions are retrieved from the server and cached and there is only a little bit of mapping to do. Enjoy it.

## Specifying a Schema ##

You can also specify a schema for the procedure that gets called:

	var beer = Database.Dynamic<Beer>().dbo.FindBeer("IPA");
	// calls dbo.FindBeer

Or...

	var beer = Database.Dynamic<Beer>().bartender.FindBeer("IPA");
	// calls bartender.FindBeer

## Async Dynamic Database Calls ##

Whoops! Almost forgot. If you append "Async" to your method name, Insight will make an async call and return a Task. Like so:

	Task<IList<Beer>> myOrder = Database.Dynamic<Beer>().FindBeerAsync("IPA");

See [[Async Commands and Queries]] for details on asynchronous calls.

## Reserved Parameters ##
When calling a DynamicConnection method, Insight reserves the following named parameters. All other parameters are passed to your stored procedure.

* cancellationToken
* commandTimeout
* returnType
* transaction
* withGraph
* withGraphs

See [[Common Method Parameters]] for details on these parameters.