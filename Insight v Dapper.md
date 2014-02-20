# Insight vs. Dapper #

NOTE: this comparison is pretty old, so some information may be incorrect.

A lot of the core bits of Insight.Database were based upon the [Dapper ORM](https://github.com/SamSaffron/dapper-dot-net). The original goal of Insight was to create a set of wrapper methods that favored the use of stored procedures rather than SQL text. In fact, early internal versions of Insight used the Dapper core code to perform its mapping. Some of this functionality could easily be written as extensions on top of Dapper, other bits cannot.

Below are some of the differences between the engines and some info about how the core engines are currently different.

## Stored Procedure Orientation ##

One of the design goals for Insight is to prefer stored procedures over SQL text. It's a small difference, but 

Insight Stored Procedure Code

	var users = c.Query<User>("GetUser", new { Id = 1 });

Dapper Stored Procedure Code

	var users = c.Query<User>("GetUser", new { Id = 1 }, commandType: CommandType.StoredProcedure);

Insight SQL Text Code

	var users = c.QuerySql<User>("SELECT * FROM Users WHERE Id = @Id", new { Id = 1 });

Dapper SQL Text Code

	var users = c.Query<User>("SELECT * FROM Users WHERE Id = @Id", new { Id = 1 });

Not much difference here, other than defaulting commandType to `CommandType.StoredProcedure` and using the `Sql` suffix to denote SQL Text evaluation.

If you are using Insight v1.1, you can make the code even a little prettier:

Insight Dynamic Stored Procedure Code

	var users = c.Dynamic<User>().GetUser(id: 1);

## Auto-Open/Close Semantics ##

I'm a big fan of deleting code for readability purposes, so I wanted to support [[Auto-Open]]/close semantics on connections. `using` statements just make things a little harder to read. So Insight supports auto-open on connections.

Insight Code

	var users = c.Query<User>("GetUser", new { Id = 1 });

Dapper Code

	using (SqlConnection c = new SqlConnection(connectionString))
	{
		c.Open();
		var users = c.Query<User>("GetUser", new { Id = 1 }, commandType: CommandType.StoredProcedure);
	}

You could probably make this work around the Dapper engine too.

## Async Support ##

As our code consumes more services, we have a development goal of moving towards more asynchronous processing. This includes using Async WCF and async SQL queries. 

But writing Async SQL queries directly against SqlCommand is really awful and tedious. The hardest part of the problem is closing the connection at the right time. Experienced developers make errors in threading and race conditions, so a library is needed.

Async SQL Command

- Look [here](http://msdn.microsoft.com/en-us/library/7b6f9k7k.aspx). (ick)

Async Insight Query

	Task<IList<User>> userTask = c.QueryAsync<User>("GetUser", new { Id = 1 });
	Task<IList<User>> userTask2 = c.Dynamic<User>().GetUserAsync(1);

Async Dapper Query

	// Not natively supported (as far as I know)

In order to get this working with the Dapper core code, you would need to split the parameter generation code apart from the result set deserializer code. This is probably the largest difference between the two systems. When the two pieces are split, there is a minor performance penalty, because the generated deserializer code evaluates the shape of the result set after the query, rather than tying the code to the query input shape.

A benefit of this split is that you can use Insight's GetReader separately from its ToList implementations. This gives you a little more flexibility when you need to deserialize objects from other non-ORM data readers.

## Streaming Objects ##

When dealing with large sets of objects, sometimes you don't want them to be in memory at the same time. Both Insight and Dapper default to reading all objects into memory, but support streaming objects.

Insight Streaming Query

	// DoStuff is called once per user. Connection is auto-open/closed
	c.ForEach<User>("GetUser", new { Id = 1 }, u => DoStuff(u));

Dapper Streaming Query

	using (SqlConnection c = new SqlConnection(connectionString))
	{
		c.Open();
		var users = c.Query<User>("GetUser", new { Id = 1 }, commandType: CommandType.StoredProcedure, buffered: false);

		// users is an IEnumerable<User> that keeps the reader open until the connection is closed
		// do stuff
	}

You could do the same in Dapper with a few more extension methods to wrap the connection lifecycle.

## Object List Parameters ##

One big difference between Insight and Dapper is the way lists of objects are handled as parameters. Because Insight prefers Stored Procedures, and prefers SQL Server, it uses Table-Valued Parameters as the main mechanism for sending lists of objects to the server:

Insight List Parameters

	// UpdateUsers proc takes one parameter: @Users [UserTableType]
	// Insight automatically determines the table type
	IEnumerable<User> users = new List<User>() { /* stuff */ };
	c.Execute("UpdateUsers", new { Users = users });

Dapper List Parameters

	// Dapper will issue one update command per object in the users enumerable
	IEnumerable<User> users = new List<User>() { /* stuff */ };
	c.Execute(@"UPDATE Users SET Name = @Name WHERE ID = @Id", users);
		
Insight doesn't (yet) support running multiple commands over a list of objects. If there is a request we could add it. 

As for Dapper, I don't know how to send a list of objects to a stored procedure. Looking at the code, I didn't see where it would auto-detect the parameter names and determine which properties should be translated into parameters.

Perhaps:

	// Dapper will issue one update command per object in the users enumerable
	IEnumerable<User> users = new List<User>() { /* stuff */ };
	c.Execute(@"UpdateUser @Name, @Id", users);

In this case, I think it would make multiple calls to the database, rather than a single call to the database. It would probably only work with a minor dependency on System.Data.SqlClient.

As an implementation note, the Insight implementation of lists is very different from the Dapper implementation. It also has an effect on the way the XML data type is handled.

## XML Object Support ##

Insight supports bi-directional mapping of XML columns:

- XML Column <=> XmlDocument
- XML Column <=> XDocument
- XML Column <=> string
- XML Column <=> DataContract-serialized object

I don't think Dapper supports this. Implementation note: the Dapper code uses the SqlDbType.Xml flag for use when processing lists of objects, so that would have to be re-worked in order to support XML types.

## Dynamic Objects ##

This is pretty much identical between Insight and Dapper. Insight has a handy "borg" operation called `Expand`:

	c.Execute("PourADraft", beer.Expand(glass));

## Object Hierarchies ##

Insight supports returning arbitrary object structures with multiple result sets, one-to-one, one-to-many mappings, etc.

As far as I know, nobody else does it as well as Insight.

Look, defining an interface that does strong-typed execution of SQL Text, returning 3 recordsets, the first is beer, the second is a one-to-many relationship of beer to glasses, and the third is a set of napkins.

	interface IBarDatabase
	{
		[Sql("SELECT * FROM Beer; SELECT b.ID, g.* FROM Glasses g JOIN RightGlass(...) ...")]
		[Recordset(0, typeof(Beer))]
		[Recordset(1, typeof(Glass), IsChild=true)]
		[Recordset(2, typeof(Napkin))]
		Results<Beer, Napkin> GetAllBeerAndGlassesAndNapkins();
	}

See [[Specifying Result Structures]].

## Dynamic Queries ##

This concept was borrowed from [Simple.Data](https://github.com/markrendle/Simple.Data), but when it's combined with the parameterization engine and the result set deserialization, it gets very powerful.

Insight Dynamic Query Invocation

	var users = c.Dynamic<User>().GetUser(id: 1);
	var users = c.Dynamic().GetUser(id: 1, type: typeof(User));

## Bulk Copy Support ##

Insight has support for SqlBulkCopy. It implements an IDataReader that it uses both to send objects as parameters as well as to stream objects to the Bulk Copy operation.

Insight Bulk Copy

	// Send objects to the database using the BULK INSERT protocol
	IEnumerable<User> users = new List<User>() { /* stuff */ };
	c.BulkCopy(@"Users", users, batchSize: 1000);


Dapper Bulk Copy

	// Dapper will issue one update command per object in the users enumerable
	IEnumerable<User> users = new List<User>() { /* stuff */ };
	c.Execute(@"UPDATE Users SET Name = @Name WHERE ID = @Id", users);


This is the same code that allows objects to be efficiently streamed to table-valued parameters. It uses a similar IL generation technique to the core Dapper code.

## IL Code ##

Both engines generate efficient IL for mapping your objects.

## Identity Inserts ##

Insight supports [[Identity Inserts]]. If your stored procedure or SQL text returns the identities of the inserted records, Insight can automatically Merge the identities into your existing objects. Use the InsertXXX methods to do it automatically or the Merge method to access the low-level functionality.

* Insight requires you to write your own SQL. You should be using Stored Procedures! :)
* Insight Insert supports sending up a TVP/IEnumerable of records, inserting them, then sending back the identities from the OUTPUT clause of your SQL statement, and mapping the identities back to the list of objects.

Dapper now supports an Insert method which automatically retrieves identity values from INSERT statements and assigns them to your objects.

* Dapper will automatically generate the SQL needed to do the insert and return the identities.
* I'm not sure how easy it is to send up a batch of objects with identity inserts and merge the IDs.

