Sometimes you just can't be bothered to make a class. Maybe it's a one-off query or a utility project. Insight supports dynamic objects for both parameters and query results. There is a little bit of overhead for using dynamic objects, but it's well worth it in the cases where you need the flexibility and readability of your code.

When returning data to your application, Insight returns a FastExpando object that is a dictionary that supports all of the interfaces needed to work properly with the C# `dynamic` keyword. For more fun things to do with FastExpando, see [[FastExpando and the Borg Method]].

When passing a dynamic object to Insight, it can be a `FastExpando` or anything derived from `System.Dynamic.DynamicObject`.

To send a dynamic object in as a parameter, just pass it in, **but you will have to cast it to object**.

	// make a dynamic object
	dynamic beer = new FastExpando();
	beer.Name = "Stella Artois";

	// extension methods cannot be dynamically dispatched, so we have to cast it to object
	// good news: it still works
	Database.Connection().ExecuteSql("UPDATE Beer Set OriginalGravity = @OriginalGravity WHERE Name = @Name", (object)beer);

To get a dynamic object back from Insight, simply omit the generic type parameter on the Query method:

	foreach (dynamic beer in Database.Connection().QuerySql("SELECT * FROM Beer WHERE Name = @Name", new { Name = "IPA" }))
	{
		beer.OriginalGravity = 4.2m;

		// extension methods cannot be dynamically dispatched, so we have to cast it to object
		// good news: it still works
		Database.Connection().ExecuteSql("UPDATE Beer Set OriginalGravity = @OriginalGravity WHERE Name = @Name", (object)beer);
	}

ForEach can also accept an action with a dynamic argument:

	Database.Connection().ForEachSql("SELECT * FROM Beer WHERE Name = @Name", new { Name = "IPA" },
		(dynamic beer) =>
		{
			beer.OriginalGravity = 4.2m;

			// extension methods cannot be dynamically dispatched, so we have to cast it to object
			// good news: it still works
			Database.Connection().ExecuteSql("UPDATE Beer Set OriginalGravity = @OriginalGravity WHERE Name = @Name", (object)beer);
		});