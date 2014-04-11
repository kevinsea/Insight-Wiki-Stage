(Why would you do this? Read: [Insight.Database: The Anti-Anti-ORM for .NET](http://code.jonwagner.com/2013/01/24/insight-database-the-anti-anti-orm-for-net/))

A best practice for coding is to create interface boundaries between parts of your systems. One place to put an interface boundary is between your business logic and the database. We encourage you to use stored procedures to achieve this, but up until now, you would still have to write the code to call into the database. 

Not any more! Insight will now generate the code to call the database for you. First, write your procedures:

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

- `void`, method name starts with "Insert/Upsert", first param is IEnumerable => InsertList
- `void`, method name starts with "Insert/Upsert", first param is updatable -> Insert
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

Note that when using Insert/Upsert, the result set will automatically be merged into the object that is the first parameter in the method call. This lets you do:

	i.InsertBeer(b);

and have the ID sent back into the object `b`.

## Parameter Mapping ##

Parameters are mapped the same way that they would be sent to the appropriate extension method. In general, object members are mapped to parameters by name, and lists of objects are sent to the database as Table-Valued parameters.

See [[Query Parameter Mapping]] for details on parameter mapping.

## Output Parameters ##

You can get output parameters back from your Stored Procedure. Just use a ref or out parameter for your method:

	public interface IBeerRepository
	{
		void UpdateBeer(string type, decimal price, out int recordsAffected);
	}

Some limitations on Output Parameters:

* Methods can only use ref/out when the parameters can be easily mapped to parameters. This will mostly affect insert/update methods:
	* InsertBeer(List<Beer> beers, out int count) is ok. 
	* InsertBeer(Beer beer, out int count) is not, because you pass in beer and count as parameters, the beer object cannot be exploded into individual parameters.
	* InsertBeer(int id, string beer, out int count) is ok, because Insight can map the function parameters to the proc parameters.

## Result Structures ##

By default, Insight can infer the structure of your recordset by the signature of your method.

* T - returns a Single<T>
* IList<T>, etc. - returns a List<T>
* Results<T1, T2> - returns two recordsets, containing T1, T2

However, if your results contain one-to-one or one-to-many mappings, you have to give Insight a hint. You do this with the `RecordsetAttribute`.

Here, we say that Recordset 0 (the first one) has a OneToOne<Beer, Glass> relationship:

	[Recordset(0, typeof(Beer), typeof(Glass))]
	List<Beer> GetBeer();

Here, we say that the second recordset has a OneToOne<Beer, Glass> relationship. Note that we don't specify the format of Recordset 0, so Insight knows that it just contains wine.

	[Recordset(1, typeof(Beer), typeof(Glass))]
	Results<Wine, Beer> GetBeer();

Here, we say that the second recordset has a one-to-many relationship. Insight will automatically read in the list of glasses and attempt to add them to the proper beer.

	[Recordset(1, typeof(Glass), IsChild=true)]
	List<Beer> GetBeer();

If you need to override the ID or Into fields, you can do that on the attribute:

	[Recordset(1, typeof(Glass), IsChild=true, ID="BeerNumber", Into="Container")]
	List<Beer> GetBeer();

In this case, the `Recordset` attribute corresponds to adding the proper `Query.Returns` statement to your call.

See [[Specifying Result Structures]] for details on how results structures are created.

## Interfaces and Transactions ##

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

## Parallel Programming ##

By default, `As<T>` will generate a single-threaded implementation. If you make calls on the interface from multiple threads, crazy things could happen (You would likely see things where the connection is closed unexpectedly or complaints that the connection is busy.)

Example:

	public async Task Multithreaded()
	{
		var ifoo = connection.As<IFoo>();

		var t1 = ifoo.FooAsync();
		var t2 = ifoo.FooAsync();
		Task.WaitAll(t1, t2);
	}

Here, we are starting two tasks calling methods on the same connection. There is nothing to prevent the system from trying to make both calls at the same time. This code will go *boom* at some point.

To fix this, tell Insight to create a multi-threaded interface implementation by calling `AsParallel<T>` instead of `As<T>`:

	public async Task Multithreaded()
	{
		// get a parallel-enabled interface
		var ifoo = connection.AsParallel<IFoo>();

		var t1 = ifoo.FooAsync();
		var t2 = ifoo.FooAsync();
		Task.WaitAll(t1, t2);
	}

A parallel interface will use a new connection for each method call. You don't have to worry about method calls conflicting anymore. The only difference is that parallel connections don't let you manage the connection lifetime with `Open` or use transactions with `OpenWithTransactionAs`.

If you want to use parallel connections in a transaction, you would need to use `System.Transactions` to create a lightweight distributed transaction. But in that case, you probably should stick to single-threaded. 

## Private Interfaces ##

If you want your interfaces and objects to be private, you will have to expose the types to the dynamic assembly that Insight creates. You can do that with the InternalsVisibleTo attribute. Add this to your assembly:

	[assembly: System.Runtime.CompilerServices.InternalsVisibleTo("Insight.Database")]

Note that this doesn't work for private interfaces nested inside of a class. It only exposes top-level classes to Insight.

## SqlAttribute ##

If you need to change the mapping between your interface method and the stored procedure, you can add the SqlAttribute to the method:

	public interface IBeerRepository
	{
		// since this is one word, Insight assumes this is a stored procedure
		[Sql("MyOtherInsertProc")]
		void InsertBeer(Beer beer);
	}

And I guess *technically* if you don't want to bother with stored procedures at all, you can call SQL text directly:

	public interface IBeerRepository
	{
		// since this is more than one word, Insight assumes this is a text call
		[Sql("SELECT * FROM Beer WHERE type = @type")]
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

If you are using SQL Schemas, you can also apply the Schema to the class:

	[Sql(Schema="MySchema")]
	public interface IBeerRepository
	{
		// this evaluates to MySchema.MyOtherInsertProc
		[Sql("MyOtherInsertProc")]
		void InsertBeer(Beer beer);
	}

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

## Abstract Class Implementation ##

Insight can also implement an abstract class for you. That way, you can create a repository object with some of your code, and still leave the boilerplate for Insight. If you pass an abstract class as the type for `.As` or `.AsParallel`, Insight will implement any unimplemented methods.

	public abstract class BeerRepository
	{
		// this is implemented by insight
		public abstract void InsertBeer(Beer beer);

		public Beer MakeBeer(string name)
		{
			var beer = new Beer(name);
			InsertBeer(beer);
			return beer;
		}
	}

	var repo = connection.AsParallel<BeerRepository>();
