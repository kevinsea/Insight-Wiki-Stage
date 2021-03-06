You can also use QueryResults with your own result classes, as long as they derive from `Results<T>`. Just pass your custom class in as the first and only generic parameter.

A handy use case for this is when you have a standard pattern for returning data, such as totals or record counts. Suppose you have the following stored procedure:

	-- NOTE: for illustration purposes. This could certainly be more efficient.
	CREATE PROC SearchBeer @Name varchar(128) AS
		SELECT * FROM Beer WHERE Name LIKE @Name
		SELECT COUNT(*) FROM Beer WHERE Name LIKE @Name

You could define your own results class where Set2 is an int and interpreted another way:

	class PageableResults<T> : Results<T, int>
	{
		public int TotalRecords { get { return Set2.FirstOrDefault(); } }
	}

Then you can use this class when returning results from the procedure:

	var pageResults = Database.Connection().QueryResults<PageableResults<Beer>>("SearchBeer", "IPA");
	Console.WriteLine ("{0} total records", pageResults.TotalRecords);

NOTE: this also works with [[Dynamic Database Calls]] if you use the returnType parameter.
