# Multiple Result Sets #

Insight supports returning multiple result sets through the QueryResults extension methods. QueryResults parses multiple result sets and returns a `Results<T1, T2...>` object.

Suppose you have the following stored procedure:

	CREATE PROC MultipleResults AS
		SELECT TOP 10 * FROM Ales
		SELECT TOP 10 * FROM Lagers

You can get the results into one object with QueryResults:

	var results = Database.Connection().QueryResults<Ale, Lager>("MultipleResults", Parameters.Empty);

	IList<Ale> ales = results.Set1;
	IList<Later> lager = results.Set2;

QueryResults currently supports up to 5 result sets in the QueryResults overrides.

## Multiple Result Sets Plus Dynamic Objects ##

If you want to return multiple result sets, and one of them should contain dynamic objects, you need to ask for a FastExpando. For example:

	var results = Database.Connection().QueryResults<FastExpando, FastExpando>("MultipleResults", Parameters.Empty);

	foreach (dynamic ale in results.Set1)
		Console.WriteLine (ale.Name);
	foreach (dynamic lager in results.Set2)
		Console.WriteLine (lager.Name);

Unfortunately, you can't use QueryResults<dynamic> because the compiler converts that to QueryResults<object> and we don't know that you want a dynamic object.

## Custom Result Objects ##

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