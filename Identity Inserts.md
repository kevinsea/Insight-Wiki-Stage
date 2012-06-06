# Identity Inserts #

The insert with identity is a very common database access pattern. Assume you have the following SQL:

	CREATE TABLE Beer ([ID] [int] IDENTITY, [Name] [varchar](128)
	GO
	CREATE PROC InsertBeer ([@Name] [varchar](128))
	AS
		-- for SQL 2008!
		INSERT INTO Beer (Name)
			OUTPUT Inserted.ID
			VALUES (@Name)

		-- for older SQL
		INSERT INTO Beer (Name) VALUES (@Name)
		SELECT ID = @@SCOPE_IDENTITY
	GO

In this case, you want to be able to insert a new record and have the generated identity returned to your object code. Insight can do this automatically for you. If your SQL returns:

* A recordset
* With the same number of rows as the number of rows inserted
* With one or more columns that can be mapped to your object (in this case ID)

Then you can use the InsertXXX methods to automatically map the results back to your objects.

	class Beer
	{
		public int ID;
		public string Name;
	}
	...
	Beer beer = new Beer() { Name = "113 IPA" };
	Connection.Insert("InsertBeer", beer);
	
	// beer.ID will automatically have the new identity assigned to it

This also works with inserting multiple records via a table-valued parameter:

	CREATE TYPE [BeerTable] AS TABLE ([Name] [varchar](128))
	GO
	CREATE PROC InsertCaseOfBeer ([@CaseOfBeer] [BeerTable] READONLY)
	AS
		INSERT INTO Beer (Name)
			OUTPUT Inserted.ID
			SELECT Name From @CaseOfBeer
	GO

Now you can send up a whole list of beer at once via the InsertListXXX methods:

	List<Beer> case = new List<Beer>()
	{
		new Beer() { "Bud Light" },
		new Beer() { "Newcastle Brown Ale" }
	};

	Connection.InsertList("InsertCaseOfBeer", case);

	// case[0].ID and case[1].ID will be assigned with the new identities

The Insert Methods are:

* Insert - insert a single record and map the results back into it
* InsertSql - SQL text version of insert
* InsertList - insert a list of objects and map the results back into the list of objects
* InsertListSql - SQL text version of InsertList

And the asynchronous versions are:

* InsertAsync
* InsertSqlAsync
* InsertListAsync
* InsertListSqlAsync
