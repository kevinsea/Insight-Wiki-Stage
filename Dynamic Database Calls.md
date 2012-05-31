# Dynamic Database Calls #

For even better syntactic sugar, try using the `.Dynamic` extension method for connections. It lets you treat stored procedures as methods on your connection:

	IList<Beer> beer = Database.Dynamic<Beer>().FindBeer("IPA");

The type parameter on Dynamic is used to determine the type of object that is deserialized from the results of the query.

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

## Async Dynamic Database Calls ##

Whoops! Almost forgot. If you append "Async" to your method name, Insight will make an async call and return a Task. Like so:

	Task<IList<Beer>> myOrder = Database.Dynamic<Beer>().FindBeerAsync("IPA");