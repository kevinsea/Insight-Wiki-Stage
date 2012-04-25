# Streaming Results Efficiently #

If you have a large set of objects being returned from the database, you may not want to keep them in memory all at once. Insight supports two ways of reading objects from the database without putting them in one giant list.

## The ForEach / ForEachSql Method ##
ForEach allows you to perform an action on each record/object returned from the database as it is deserialized.

	Database.Connection().ForEach<Beer>(
		"FindBeer", 
		new { Name = "IPA" },
		beer => Drink(beer));

You can also use dynamic objects:

	Database.Connection().ForEachSql(
		"SELECT Name, Flavor FROM Beer", 
		Parameters.Empty,
		(dynamic beer) =>
		{
			beer.Price = 0.0m;
			Console.WriteLine("YUM. Beer is free");
		});

## AsEnumerable Extension ##
This is the hard way. Well, not really that hard. But it gives you more control over your objects. The AsEnumerable<T> extension converts objects as they are read and does not put them in a list.

	using (SqlConnection connection = Database.Open())
	using (var reader = connection.GetReaderSql("SELECT * FROM Beer", Parameters.Empty))
	{
		foreach (Beer beer in reader.AsEnumerable<Beer>())
		{
			Drink(beer);
		}
	}

This also works with dynamic objects:

	using (SqlConnection connection = Database.Open())
	using (var reader = connection.GetReaderSql("SELECT * FROM Beer", Parameters.Empty))
	{
		foreach (dynamic beer in reader.AsEnumerable())
		{
			beer.Price = 0.0m;
			Console.WriteLine("YUM. Beer is free");
		}
	}
