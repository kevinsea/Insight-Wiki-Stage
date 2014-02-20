Insight can automatically convert query results to objects. Just use the Query extension method to execute a query and return a list of objects.

	class Beer
	{
		public string Name { get; internal set; }
		public string Flavor { get; internal set; }
		public decimal? OriginalGravity { get; internal set; }
	}

	IList<Beer> beer = Database.Connection().Query<Beer>("FindBeer", new { Name = "IPA" });

Look, ma! No mapping files!

The generic type argument specifies the expected return type. Insight automatically creates a mapping between the schema of the result set and the object type and performs the mapping. For more details see [[Mapping Results to Objects]].

This also works with SQL text:

	IList<Beer> beer = Database.Connection().QuerySql<Beer>(
		"SELECT * FROM Beer WHERE Name = @Name",
		new { Name = "IPA" });

If you already have an IDataReader (from any source!), you can use the ToList extension to convert the reader to objects:

		using (SqlConnection connection = Database.Open())
		using (IDataReader reader = connection.GetReader("FindBeer", new { Name = "IPA" }))
		{
			IList<Beer> beer = reader.ToList<Beer>();
		}

Or AsEnumerable<T>:

		using (SqlConnection connection = Database.Open())
		using (IDataReader reader = connection.GetReader("FindBeer", new { Name = "IPA" }))
		{
			foreach (Beer beer in reader.AsEnumerable<Beer>())
			{
				// drink?
			}
		}

[[Executing SQL Commands]] - BACK || NEXT- [[Identity Inserts]]
