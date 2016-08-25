Note this example never sets Beer.ID for new rocords. **Jon what am I doing wrong?**

Let's create a very simple example that puts all of the code together and works right out of the box. 

First, let's create a table and some stored procedures and some test data:

```SQL
CREATE TABLE Beer (
	  [ID] int IDENTITY(1,1) PRIMARY KEY
	, [Type] varchar(128)
	, [Description] varchar(128)) 
GO

CREATE PROC InsertBeer @type varchar(128), @description varchar(128) 
AS
	INSERT INTO Beer ([Type], [Description]) OUTPUT inserted.ID
		VALUES (@type, @description)
GO
        
CREATE PROC GetBeerByType @type varchar(128) 
AS 
	SELECT * FROM Beer WHERE Type = @type 
GO

-- Add some data
EXEC InsertBeer 'IPA', 'Harpoon IPA';
EXEC InsertBeer 'IPA', 'Heady Topper';
EXEC InsertBeer 'Swill', 'Pabst Blue Ribbon';
```

1. Create a Console Application project called 'Insight_HelloWorld' 

1. Add Insight.Database (see [[Installing-Insight]])

1. Paste this code into program.cs:

``` C#
using System;
using System.Collections.Generic;
using System.Data.SqlClient;
using Insight.Database;

namespace Insight_HelloWorld
{
	class Program
	{
		static void Main(string[] args)
		{
			var repo = new BeerRepository();
			IList<Beer> ipas = repo.GetBeersByType("IPA");

			var newBeer = new Beer() { Type = "Pale Ale", Description = "Test" };

			repo.InsertBeer(newBeer);
			IList<Beer> paleAles = repo.GetBeersByType("Pale Ale");
		}
	}

	class BeerRepository
	{
		private string _connectionString = Stuff.GetConnectionStr(); //return @"server=...";

		public IList<Beer> GetBeersByType(string type)
		{
			var parms = new GetBeersByTypeParams() { Type = type };
			var connection = new SqlConnection(_connectionString);
			IList<Beer> result = connection.Query<Beer>("GetBeerByType", parms);
			return result;
		}

		public void InsertBeer(Beer beer)
		{
			var connection = new SqlConnection(_connectionString);
			var res = connection.Execute("[InsertBeer]", beer);
		}

	}

	internal class GetBeersByTypeParams { internal string Type; }

	public class Beer
	{
		public int ID { get; set; }
		public string Type { get; set; }
		public string Description { get; set; }
	}
}
```
1. Set your connection string in the BeerRepository class
1. That's it!  Set a breakpoint at the end of main and run the code

Not sure how it all works?  The Connections, Execute, and Getting Results sections explain it all.

Want to do this with even less code,  read [[https://github.com/kevinsea/wiki-test/wiki/Auto-Interface-Implementation]]
