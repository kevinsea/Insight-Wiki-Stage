Lists of objects and SQL have never gone well together. To query for a set of objects, you would have to encode your ID list into a string or XML and decode it in your stored procedure. To insert a list of objects, you would have to do individual inserts. Not anymore.

## Sending a List of ValueType to SQL Text ##

This is a common case. You want to query for a list of objects by ID. When using SQL text commands, Insight can transform an `IEnumerable<ValueType>` to an IN(...) portion of a WHERE clause:

	IEnumerable<String> names = new List<String>() { "Sly Fox IPA", "Hoppapotamus" };
	var beer = Database.Connection().QuerySql("SELECT * FROM Beer WHERE Name IN (@Name)", new { Name = names });

Insight will convert this to:

	SELECT * FROM BEER WHERE Name IN (@Name1, @Name2)

NOTE: This is limited to the maximum number of parameters your SQL provider allows.

This also works with arrays:

	IEnumerable<String> names = new String[] { "Sly Fox IPA", "Hoppapotamus" };
	var beer = Database.Connection().QuerySql("SELECT * FROM Beer WHERE Name IN (@Name)", new { Name = names });

## Sending a List of ValueType to a Stored Procedure ##

Stored procedures don't allow for a dynamic parameter list, so we can't use the same technique as above. However, some databases support table types and are is nice enough to let us query a stored procedure to get those table types. So let's say we have a proc like this:

	CREATE TYPE BeerNameTable AS TABLE (Name [nvarchar](128))
	GO

	CREATE PROCEDURE GetBeer (@Names [BeerNameTable] READONLY)
	AS
		SELECT * FROM Beer WHERE Name IN (SELECT Name FROM @Names)
	GO

With Insight, we can just call it like this:

	IEnumerable<String> names = new List<String>() { "Sly Fox IPA", "Hoppapotamus" };
	var beer = Database.Connection().Query("GetBeer", new { Name = names });

Or like this...Insight assumes that if you pass an enumerable as the parameter, then it should convert it to a table-valued parameter and send it to the stored procedure.

	var beer = Database.Connection().Query("GetBeer", names);


Insight will automatically convert `IEnumerable<ValueType>` to the first column of a table-valued parameter.

## Sending a List of Objects to a Stored Procedure ##

But what if you have a list of *objects* to send to the database? No problem. 

First, you need a user-defined table type and a stored procedure that accepts the table type:

	CREATE TYPE BeerTable AS TABLE (Name [nvarchar](128), Flavor [nvarchar](128), OriginalGravity [decimal](18,2))
	GO

	CREATE PROCEDURE UpdateBeer (@Beer [BeerTable] READONLY)
	AS
		UPDATE Beer
			SET Flavor = up.Flavor, OriginalGravity = up.OriginalGravity
			FROM @Beer up
			WHERE Beer.Name = up.Name
	GO

Now you can send in a lot of updates in a batch:

	List<Beer> beer = new List<Beer>();
	beer.Add(new Beer() { Name = "Sly Fox IPA", Flavor = "yummy", OriginalGravity = 4.2m });
	beer.Add(new Beer() { Name = "Hoppopotamus", Flavor = "hoppy", OriginalGravity = 3.0m });

	Database.Connection().Execute("UpdateBeer", new { Beer = beer });

The list of objects is automatically mapped to the table-valued-parameter.

This works because the database is kind enough to tell us the number and type of the parameters for the stored procedure. Are ya diggin' stored procedures yet?

## Sending a List of Objects to SQL Text ##

If you don't have a stored procedure, you can still use table-valued parameters.

Since we don't have any meta-data on your query, Insight will look for a table type with the same name as the contents of the list.

For example:

	List<Beer> beer = new List<Beer>();
	Database.Connection().ExecuteSql("INSERT INTO Beer SELECT * FROM @Beer", new { Beer = beer });

At runtime, Insight uses reflection to determine that the Beer field on the anonymous type is an IEnumerable<Beer>. It then tries to convert that parameter to a [BeerTable]. Note that it adds "Table" to the end of the table type name.

## Empty Lists ##

Unlike most types of parameters, SQL server doesn't let you have NULL table-valued parameters. If you omit a TVP, SQL will use an empty table. This makes it really easy to forget a TVP when calling a procedure.

To help you remember, Insight will throw an exception if you pass a NULL list to a TVP. You should always specify at least an empty list. There is a handy method to get an empty list of a given type:

	Parameters.EmptyListOf<T>();

Example:

	connection.ExecuteSql("INSERT INTO Beer SELECT * FROM @Beer", Parameters.EmptyListOf<Beer>());


[[Identity Inserts]] - BACK || NEXT- [[Transactions]]