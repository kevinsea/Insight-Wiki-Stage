# Multiple Result Sets #

If you have a query that returns multiple result sets, you will need to do a little bit of work.

1. Use GetReader to get a reader from the database.
2. Use ToList or AsEnumerable to get the list of objects from the reader.

	using (SqlConnection connection = Database.Open())
	using (var reader = connection.GetReaderSql("SELECT * FROM Beer SELECT * FROM Glasses", Parameters.Empty))
	{
		var beer = reader.ToList<Beer>();
		var glasses = reader.ToList<Glass>();
	}

Note that ToList (and AsEnumerable) automatically advance to the next recordset after consuming the objects.