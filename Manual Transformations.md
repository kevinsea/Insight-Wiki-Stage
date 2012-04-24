# Manual Transformations #

Sometimes you need to do your own data transformations from the result set to your objects. Maybe Insight doesn't map it right for you. Maybe you need to do calculations before returning the data. Maybe you just want to be cool.

You probably won't need this often, but when you do, it's very handy.

You can use the Insight query engine and auto-open semantics and still map your own data. The Query methods have a version that takes a read function where you can manually transform the data.

		List<Beer> beer = Database.Connection().Query(
			"FindBeer",
			new { Name = "IPA" },
			reader =>
			{
				List<Beer> b = new List<Beer>();

				while (reader.Read())
				{

					// do whatever you want here

					b.Add(new Beer(reader["Name"].ToString()));
				}

				return b;
			});

Within the read action, you can do calculations, take summaries, or perform other actions.

		int totalNameLength = Database.Connection().Query(
			"FindBeer", 
			new { Name = "IPA" }, 
			reader => 
				{
					int total = 0;
					while (reader.Read())
					{
						total += reader["Name"].ToString().Length;
					}
					return total;
				});
			
		Console.WriteLine(totalNameLength);

If you would rather act on the reader yourself and manage the lifetime of the connection, you can use the GetReader method:

		using (SqlConnection connection = Database.Open())
		using (IDataReader reader = connection.GetReader("FindBeer", new { Name = "IPA" }))
		{
			while (reader.Read())
			{
				// do stuff with the reader here
			}
		}
