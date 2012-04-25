# FastExpando and Mutations #

Sometimes you have an ugly database that you aren't allowed to modify. It may be ugly, AND you aren't allowed to create new views or stored procedures to make it look nice. But you want your code to look nice, right?

You have two choices:

1. Make a class, bind the data to private members and expose pretty properties.
1. You don't want a class, so use FastExpando.

The FastExpando supports two methods to help your code look nicer - Mutate and Transform. Both of them accept a map (a dictionary of string to string), and convert the field names into new field names.

* `expando.Mutate(map)` - modifies the current `expando` in-place
* `dynamic newexpando = expando.Transform(map)` - transforms an expando into a new expando

Usually you will have the mapping as a static dictionary that is reused across calls.

In all cases, unmapped properties are retained as-is.

NOTE: if you map two source properties to the same target value, the behavior is undefined.

## FastExpando.Mutate ##

			var mapping = new Dictionary<string, string>() { { "Name", "TheName" } };

			dynamic beer = Database.Connection().QuerySql("SELECT * FROM Beer WHERE Name = @Name", new { Name = "IPA" }).First();
			beer.Mutate(mapping);
			Console.WriteLine(beer.TheName);

## FastExpando.Transform ##

			dynamic beer = Database.Connection().QuerySql("SELECT * FROM Beer WHERE Name = @Name", new { Name = "IPA" }).First().Transform(mapping);
			Console.WriteLine(beer.TheName);

## Transforming a List ##

IEnumerable<FastExpando> also supports transforming the objects while enumerating:

			foreach (dynamic beer in Database.Connection().QuerySql("SELECT * FROM Beer WHERE Name = @Name", new { Name = "IPA" }).Transform(mapping))
				Console.WriteLine(beer.TheName);