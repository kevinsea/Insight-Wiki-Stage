# Proposed v4.0 Changes #

**THIS IS A DRAFT ACCEPTING COMMENTS**

To comment, edit this wiki page and put your comment in a quote box with your name/ID:

> Jon - This is awesome!

> lobetuf - definitely awesome!  

For v4.0, I'm looking at one major goal: 

	Making it easy to return arbitrary trees of objects from SQL results.

As usual, the design goals are:

* Be Productive - easy tasks should be easy.
* Be Natural - code should be written in its natural language. (SQL is SQL. Code is Code.)
* Be Simple - if it gets too icky, you'll pay for it later. 
* Be Fast - high-performance is key. All of this will be runtime-compiled into fast code. I promise.

Since this will be a major release, I'll be allowing breaking changes in order to make v4.0 "Be Natural". If things are going to break, my goals will be to: 

* Have any breaking changes detected by compilation.
* Have an easy upgrade path (e.g. rename a class or member).
* Support a "Compatibility" assembly for deprecated methods that I'd like to get rid of, but you might not have time to update/test your code.

Below is the proposed documentation for v4.0, as I expect it to end up. (I haven't written the code yet, so there will probably be changes.) Differences from v3.x will be noted.

Reader Notes:

* For clarity, much of the SQL below has been truncated.
* All of the QuerySql methods have a corresponding Query method that takes a proc name rather than SQL.

# Returning Objects #

When you return data from a Query, Insight converts the results to a list of objects.

By default, Query<T> will return a list of T.

	List<Beer> yum = c.QuerySql<Beer>("SELECT * FROM Beer");

# One-to-One Mappings #

If you are returning a 1-1 set of objects in a single recordset, you can add the additional classes in the template list. Insight will automatically assign the proper fields to the objects and assemble your objects.

**TODO: re-explain how it does this. Field mapping/object detection is not expected to change.**

	class Beer
	{
		public Glass Glass; // one proper glass per beer
	}

	List<Beer> yum = c.QuerySql<Beer, Glass>("SELECT * FROM Beer JOIN Glasses");

You can also tell Insight that whenever you return a Beer, you will also generally return a Glass. Do this by adding a Recordset attribute.

**CHANGE FROM v3: this used to be the DefaultGraph attribute. You should be able to easily replace or use a compatibility layer.**

	[Recordset(typeof(Graph<Beer, Glass>))]
	class Beer { ... }

Now, whenever Insight reads a Beer, it will also look for a Glass unless you tell it otherwise. If there is no Glass in the recordset, then the Glass property will be null.

The other option is to pass a recordset definition into the query. Below, we tell Insight that the recordset contains a Beer and a Glass.

**CHANGE FROM v3: this used to be the `returnType` or `withGraph` parameter and would take a `typeof(Graph<T>)`. The compatibility layer will still support these versions.**

	List<Beer> yum = c.QuerySql("SELECT * FROM Beer JOIN Glasses",
		returns: () => Results.Recordset<Beer, Glass>());

Design Notes:

* Be Natural - `QuerySql<T...>` is the most natural expression of this concept. So it stays.
* Be Natural - `returnType` and `withGraph` aren't as intuitive as `returns` and `Results.Recordset`, so I'm changing the names.
* Be Fast - The returns parameter is a Function so that it is only called once when Insight does the initial mapping.
* Be Productive - When using the returns parameter construct, the compiler will automatically detect the return type from the signature of this parameter.
* Be Productive - The Query methods have a lot of optional parameters, so you generally will need to specify the `returns:` label. Most likely its position will replace `returnType` or `withGraph` so you won't need to change your code much.
* Be Productive - The returns parameter is a great way to let you tell Insight how your data is structured. Let's build on that concept below. 

# Multiple Unrelated Recordsets #

If you want to return multiple recordsets, Insight has built-in support:

	Results<Beer, Glass> results = c.QueryResultsSql<Beer, Glass>(
		"SELECT * FROM Beer; SELECT * FROM Glasses");

In this case, `results.Set1` has your beer and `results.Set2` has your glasses.

You could also do this with:

	Results<Beer, Glass> results = c.QueryResultsSql<Results<Beer, Glass>>(
		"SELECT * FROM Beer; SELECT * FROM Glasses");

Or by using the returns parameter:

	Results<Beer, Glass> results = c.QuerySql(
		"SELECT * FROM Beer; SELECT * FROM Glasses",
		returns: () => Results
				.Recordset<Beer>()
				.Then<Glass>());

You can have more than two recordsets returned:

	Results<Beer, Glass, Napkin> results = c.QuerySql(
		"SELECT * FROM Beer; SELECT * FROM Glasses; SELECT * FROM Napkins",
		returns: () => Results
				.Recordset<Beer>()
				.Then<Glass>()
				.Then<Napkin>());

Now, `results.Set3` contains napkins.

**CHANGE FROM v3: by replacing `returnType` and `withGraph` with the `returns` parameter, we will be able to support arbitrary structures.**

Design Notes:

* Be Natural - personally, I like the `QueryResultsSql<T...>` and the `returns:` methods the most.
* Be Productive - The `returns:` method can also be extended to handle one-to-many relationships.

# One-to-Many Mappings #

Sometimes when you return multiple recordsets, what you want to do is return a tree of objects in a one-to-many relationship. Let's say there is more than one appropriate glass for a given kind of beer. Each beer (one) can have many glasses.

	class Beer
	{
		public int ID;
		public ICollection<Glass> Glass; // multiple proper glass per beer
	}

**QUESTION: What types should this support? IEnumerable? IList? ICollection?**
> lobetuf - I prefer ICollection<T>.  While most times, IEnumerable<T> would be sufficient to do read-only work and iterate over the result set, I utilize ICollection<T> in my POCOs to allow modifying my collections before iterating and persisting back

**QUESTION: Who is responsible for allocating the collection? Your code or Insight? Or either?**
> lobetuf - I'd say that when returning results, Insight should allocate it; when I'm submitting data to be persisted, I should allocate it.   

You would probably return recordsets like this:

	SELECT * FROM Beer WHERE Type="IPA"
	SELECT b.ID, g.* FROM Glasses g JOIN RightGlass(...) ...

Now, we want to roll up the glasses into the Glass collection on Beer:

	List<Beer> yum = c.QuerySql<Beer>(sql,
		returns: Results
					.Returns<Beer>()
					.ThenChild<Glass>());

Here, we're telling Insight to expect two recordsets. The first one contains Beer. The second one is expected to be a set of Glasses. Insight will attempt to automatically add them to an `ICollection<Glass>` on Beer.

## How Insight will handle the one-to-many mapping ##

By default, Insight will look for the first field in the Beer class with "ID" at the end of the name. It will use that as the parent key field. It will also assume that the first field in the recordset is the reference to the parent. It will then automatically put the right glasses into the right beer.

If Insight doesn't pick the right ID field, you can specify it:

		returns: Results
					.Returns<Beer>(id: b => b.MyIdentifier)
					.ThenChild<Glass>());

Or to do it in one place, add it to the `Recordset` attribute on your class:

	[Recordset(ID = "MyIdentifier")]
	public class Beer
	{
		public int MyIdentifier;
		...
	}

If Insight doesn't pick the right target field, you can specify it:

		returns: Results
					.Returns<Beer>()
					.ThenChild<Glass>(into: b => b.BarShelf));

**QUESTION: for the one-to-many case, is there any reason why we can't assume the first column in a child recordset refers to the parent's ID?**

> lobetuf - I think it's fair to impose such a constraint. 

# Multiple One-to-Many Mappings #

You can also return more than two recordsets. In this case Napkins go into Beer too:

		returns: Results
					.Returns<Beer>()
					.ThenChild<Glass>())
					.ThenChild<Napkin>());

Subsequent recordsets can have their own one-to-one relationships:

		returns: () => Results
					.Returns<Beer>()
					.ThenChild<Glass, Napkin>());

You can also create multiple levels. Here we put the napkins under the glasses instead of the beer:

		returns: () => Results
					.Returns<Beer>()
					.ThenChild<Glass>())
					.ThenChild<Napkin>(into: b => b.Glasses.Into().Napkins));

Note that if Glass has a funky ID field, you would specify it this way: 

		returns: () => Results
					.Returns<Beer>()
					.ThenChild<Glass>(id: g => g.MyIdentifier))
					.ThenChild<Napkin>(into: b => b.Glasses.Into().Napkins));

Design Notes:

* Be Natural - the fluent-style specification seems to feel right. Comments on the names are appreciated.
* Be Productive - rather than strings, the `id` and `into` parameters are functions so you get compile-time checking of your mappings. Also refactor support.
* Be Simple - when specifying the return structure, the configuration feels simple and also stable (meaning I don't expect a lot of additional feature requests).

# Mix and Match Your Mappings #

Okay, so this is getting a little crazy, but it shouldn't be too icky. Returning 4 recordsets, with two parents and two children!

	Results<Beer, Wine> results = c.QuerySql(
		@"	SELECT * FROM Beer WHERE Type="IPA"
			SELECT b.ID, g.* FROM Glasses g JOIN RightGlass(...) ...
			SELECT * FROM Wine WHERE Type="IPA"
			SELECT w.ID, g.* FROM Glasses g JOIN RightGlass(...) ...", 
		returns: () => Results
					.Returns<Beer>()
					.ThenChild<Glass>()
					.Then<Wine>()
					.ThenChild<Glass>();

Beer and Wine are the parents. Both of them have glasses as children.

Design Notes:

* Be Natural - the results definition matches the recordsets.
* Be Productive - note that other than that, the column mappings are all automatic.

# Fluent Configuration #

Perhaps you don't want to put any SQL junk in your objects. You would rather do it in a single place. If you prefer, you can do fluent configuration:

Define the ID of the Beer class:

	Insight.Configure<Beer>(id: b => b.MyIdentifier);

Set the default one-to-one mapping for the Beer class: 

	Insight.Configure<Beer>().Recordset<Beer, Glass>();

Set the results of the "GetMenu" procedure:

	Insight.Procedure("GetMenu")
					.Returns<Beer>()
					.ThenChild<Glass>()
					.Then<Wine>()
					.ThenChild<Glass>();

NOTE: you wouldn't do this for inline SQL. If you're doing inline SQL, you should probably put the mapping there.

Design Notes:

* Be Natural - this uses the same code that you would write inline.
* Be Fast - don't worry, it will be fast. 

# Interface Configuration #

If I haven't jumped up and down enough about how awesome auto-interfaces are, let me do it now (jump jump jump).

You can define the interface to your database simply by declaring an interface:

	interface IBarDatabase
	{
		Beer GetBeerById(int id);
		IList<Beer> GetBeerByType(string type);
	}

	IBarDatabase db = connection.As<IBarDatabase>();

For one-to-one relationships, we can add a `Recordset` attribute to the method:

	interface IBarDatabase
	{
		[Recordset(typeof(Beer), typeof(Glass))]
		IList<Beer> GetBeerByType(string type);
	}

**CHANGE FROM V3: this used to be the `DefaultGraph` attribute**

We want to be able to do the same recordset mapping for interfaces. Multiple results is pretty easy:

	interface IBarDatabase
	{
		[Sql("SELECT * FROM Beer; SELECT * FROM Glass")]
		Results<Beer, Glass> GetAllBeerAndGlasses();
	}

(Oh yeah, here we did inline SQL.)

Note how Insight detects that you have two recordsets because you declared the return type to be `Results<Beer, Glass>`. Unfortunately, that doesn't work for arbitrary trees, so we're going to have to use attributes.

By adding multiple `Recordset` attributes, we can tell Insight what to expect.

Let's try a one-to-many relationship:

	interface IBarDatabase
	{
		[Sql("SELECT * FROM Beer; SELECT b.ID, g.* FROM Glasses g JOIN RightGlass(...) ...")]
		[Recordset(typeof(Beer))]
		[Recordset(typeof(Glass))]
		IList<Beer> GetAllBeerAndGlasses();
	}

If you needed to override the `id` and `into` properties:

	interface IBarDatabase
	{
		[Sql("SELECT * FROM Beer; SELECT b.ID, g.* FROM Glasses g JOIN RightGlass(...) ...")]
		[Recordset(typeof(Beer), ID="MyIdentifier")]
		[Recordset(typeof(Glass), Into="BarShelf")]
		IList<Beer> GetAllBeerAndGlasses();
	}

Three levels would be:
 
	interface IBarDatabase
	{
		[Sql("SELECT * FROM Beer; SELECT b.ID, g.* FROM Glasses g JOIN RightGlass(...) ...")]
		[Recordset(typeof(Beer))]
		[Recordset(typeof(Glass))]
		[Recordset(typeof(Napkin), Into="Glasses.Napkin")]
		IList<Beer> GetAllBeerAndGlassesAndNapkins();
	}

Note that here we told Insight to put the Napkins inside the Glasses.

Mixing up one-to-many and one-to-many (Beer can go in many glasses, Wine snobs only use one particular glass):

	interface IBarDatabase
	{
		[Recordset(typeof(Beer))]
		[Recordset(typeof(Glass))]
		[Recordset(typeof(Wine), typeof(Glass))]
		Results<Beer, Wine> GetBeerAndWine();
	}

Mixing up unrelated and one-to-many:

	interface IBarDatabase
	{
		[Sql("SELECT * FROM Beer; SELECT b.ID, g.* FROM Glasses g JOIN RightGlass(...) ...")]
		[Recordset(typeof(Beer))]
		[Recordset(typeof(Glass))]
		[Recordset(typeof(Wine))]
		[Recordset(typeof(Glass))]
		Results<Beer, Wine> GetBeerAndWine();
	}

Because we returned `Results<Beer, Wine>`, Insight knows that recordsets 1 and 3 are top level, and 2 and 4 are children of the previous parents.

But what if we want to have the same type as a parent and as a child. Well, we'll have to hint to Insight.

	interface IBarDatabase
	{
		[Sql("SELECT * FROM Beer; SELECT b.ID, g.* FROM Glasses g JOIN RightGlass(...) ...")]
		[Recordset(typeof(Beer))]
		[Recordset(typeof(Glass), IsChild = true)]
		[Recordset(typeof(Glass), IsParent = true)]
		Results<Beer, Glass> GetBeerAndEvenWrongGlasses();
	}

If there is some confusion and you don't set IsChild or IsParent, Insight will politely throw an exception.

Note that if you are using stored procedures, you don't have to use attributes. You can use the fluent configuration described above:

	interface IBarDatabase
	{
		Results<Beer, Wine> GetBeerAndWine();
	} 

	// somewhere else....
	Insight.Procedure("GetBeerAndWine")
		.Returns<Beer>()
		.ThenChild<Glass>()
		.ThenChild<Wine>()
		.ThenChild<Glass>();

Design Goals:

* Be Productive - encourage people to use auto-interfaces by making it just as easy to do multiple results.
* Be Productive - let people code with attributes or configuration.
* Be Simple - no I won't write something that reads XML files and configures things. I hate XML configuration.