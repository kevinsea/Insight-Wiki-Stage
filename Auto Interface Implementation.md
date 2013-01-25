# Auto Interface Implementation #

(Why would you do this? Read: [Insight.Database: The Anti-Anti-ORM for .NET](http://code.jonwagner.com/2013/01/24/insight-database-the-anti-anti-orm-for-net/))

A best practice for coding is to create interface boundaries between parts of your systems. One place to put an interface boundary is between your business logic and the database. We encourage you to use stored procedures to achieve this, but up until now, you would still write the code to call into the database. 

Not any more! Insight will now generate the code to call the database for you. Write your procedures:

	CREATE PROC InsertBeer @type varchar(128), @description varchar(128) AS
		INSERT INTO Beer (Type, Description) OUTPUT inserted.ID
			VALUES (@type, @description)
	GO
	CREATE PROC GetBeerByType @type [varchar] AS SELECT * FROM Beer WHERE Type = @type 
	GO

Create an interface to match your stored procedures:

	public interface IBeerRepository
	{
		void InsertBeer(Beer beer);
		IList<Beer> GetBeerByType(string type);
	}

Then you convert your DbConnection, DbConnectionStringBuilder or ConnectionStringSettings to your interface with the As extension methods:

	DbConnection c = new SqlConnection(connectionString);
	IBeerRepository i = c.As<IBeerRepository>();

Insight implements the interface for you automatically, and you can just call the method, with full type-safety and full performance of IL-generated code:

	i.InsertBeer(b);
	var results = i.GetBeerByType("ipa");

## Method Mapping ##

When generating the implementation of the interface, Insight uses the DbConnection extension methods. It maps the interface methods by the return type using the following rules:

- `void`, method name starts with "Insert/Update/Upsert", first param is IEnumerable => InsertList
- `void`, method name starts with "Insert/Update/Upsert", otherwise => Insert
- `void`, otherwise => Execute
- `IList<T>` => Query
- `Results<>` => QueryResults
- other type, primitive type => ExecuteScalar
- other type, otherwise => Single

For methods that return Task, Insight uses the same rules as above, but the methods are called asynchronously:

- `Task` => same as `void`
- `Task<IList<T>>` => same as `IList<T>`
- `Task<Results<>>` => same as `Results<T>`
- `Task<other type>` => same as other type

For asynchronous methods, Insight will also automatically remove the word "Async" from the end of your method name when mapping to the name of the stored procedure.

## Parameter Mapping ##

Parameters are mapped the same way that they would be sent to the appropriate extension method. In general, object members are mapped to parameters by name, and lists of objects are sent to the database as Table-Valued parameters.

See [[Query Parameter Mapping]] for details on parameter mapping.

## Result Mapping ##

See [[Mapping Results To Objects]] for details on how results are mapped back to objects.

## Even More Convenient Interfaces ##

The class that Insight generates also supports IDbConnection and IDbTransaction, so you can add those interfaces to your interface like this:

	public interface IBeerRepository : IDbConnection, IDbTransaction
	{
		// blah
	}

Then you can use the object in your using statements like this:

	// this opens the connection and starts a transaction
	// the using statement will dispose the connection automatically for you
	using (var repo = connectionString.OpenWithTransactionAs<IBeerRepository>())
	{
		repo.InsertBeer(b1);
		repo.InsertBeer(b2);

		// this is called on IDbTransaction
		repo.Commit();
	}

I don't think you can make transactional calls to the database with less code than this.

## Private Interfaces ##

If you want your interfaces and objects to be private, you will have to expose the types to the dynamic assembly that Insight creates. You can do that with the InternalsVisibleTo attribute. Add this to your assembly:

	[assembly: System.Runtime.CompilerServices.InternalsVisibleTo("Insight.Database")]

Note that this doesn't work for private interfaces nested inside of a class. It only exposes top-level classes to Insight.

## SqlAttribute ##

If you need to change the mapping between your interface method and the stored procedure, you can add the SqlAttribute to the method:

	public interface IBeerRepository
	{
		[Sql("MyOtherInsertProc")]
		void InsertBeer(Beer beer);
	}

And I guess *technically* if you don't want to bother with stored procedures at all, you can call SQL text directly:

	public interface IBeerRepository
	{
		[Sql("SELECT * FROM Beer WHERE type = @type", CommandType.Text)]
		IList<Beer> GetBeerByType(string type);
	}

Because, underneath it all, Insight is creating a class like this:

	class Anonymous : DbConnectionWrapper, IBeerRepository
	{
		public IList<Beer> GetBeerByType(string type)
		{
			return InnerConnection.QuerySql("SELECT * FROM Beer WHERE type = @type", new { type = type });
		}
	}

But seriously, why should you have to write that code at all?

## Special Parameters ##

In some cases, you may need to control some of the other [[Common Method Parameters]] like `commandTimeout` dynamically. If you add a parameter that matches the common parameter, Insight will pass that along to the framework rather than to the stored procedure.

NOTE: both the name *and* the type must match:

	public interface IBeerRepository
	{
		// commandTimeout is passed to the framework
		void InsertBeer(Beer beer, int? commandTimeout);
	
		// cancellationToken is passed to the framework
		Task GetBeerByType(string type, CancellationToken? cancellationToken);
	}