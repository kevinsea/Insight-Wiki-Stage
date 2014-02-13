# Insight.Database 4.0.0-alpha1 is now on NuGet - 2/12/2014  #

The code is also in the 4.0 branch if you want to look at it.

**THIS IS A DRAFT ACCEPTING COMMENTS - Feel free to add them.**

To comment, edit this wiki page and put your comment in a quote box with your name/ID.

(I've taken out the last round of comments. Feel free to put more in.)

> Jon - This is awesome!

For v4.0, I'm looking at one major goal: 

	Making it easy to return arbitrary trees of objects from SQL results.

As usual, the design goals are:

* Be Productive - easy tasks should be easy.
* Be Natural - code should be written in its natural language. (SQL is SQL. Code is Code.)
* Be Simple - if it gets too icky, you'll pay for it later. 
* Be Fast - high-performance is key. All of this will be runtime-compiled into fast code. I promise.

Since this is a major release, I've allowed some features to be removed/broken so that the 4.x API is nice and clean. The good news is that there will be a v3.x compatibility library that you can drop in to get almost all of the functionality back. 

Reader Notes:

* For clarity, much of the SQL below has been truncated.
* All of the QuerySql methods have a corresponding Query method that takes a proc name rather than SQL. And Async, of course.

## Changes from v3.x ##

* DefaultGraphAttribute has been replaced by the RecordsetAttribute
* Extension methods that used to take a withGraph(s) parameter now take a returns parameter.

## Not Yet in v4.0-alpha1 ##

* v3.x Compatibility Library - this will be in alpha2.
* Fluent Configuration
* More than two levels of children on an interface.

## Upgrading from Previous Versions ##

* If you didn't use the Graph class, v4.x should be a drop in replacement.
* If you did use the Graph class, then you can make a few changes or wait for the 3.x Compatibility Library in 4.0.0-alpha2.

**I'd like to know what things don't compile without changes on 4.x, so if you drop it in and things don't work, please let me know.**

# Returning Objects #

When you return data from a Query, Insight converts the results to a list of objects.

By default, `Query<T>` will return a list of T.

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


	[Recordset(typeof(Beer), typeof(Glass))]
	class Beer { ... }


Now, whenever Insight reads a Beer, it will also look for a Glass unless you tell it otherwise. If there is no Glass in the recordset, then the Glass property will be null.

**CHANGE FROM v3: this used to be the DefaultGraph attribute.**

The other option is to pass a recordset definition into the query. Below, we tell Insight that the recordset contains a Beer and a Glass.

	List<Beer> yum = c.QuerySql("SELECT * FROM Beer JOIN Glasses", parameters,
		Query.Returns(OneToOne<Beer, Glass>.Records));

The design of the `Query.Returns` extension methods automatically let the compiler determine the return type of QuerySql. 

**CHANGE FROM v3: this used to be the `returnType` or `withGraph` parameter and would take a `typeof(Graph<T>)`. The compatibility layer will still support these versions.**

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
		"SELECT * FROM Beer; SELECT * FROM Glasses", parameters,
		Query.Returns(Some<Beer>.Records)
			.Then(Some<Glass>.Records);

With any of the methods, You can have more than two recordsets returned:

	Results<Beer, Glass, Napkin> results = c.QuerySql(
		"SELECT * FROM Beer; SELECT * FROM Glasses; SELECT * FROM Napkins", parameters,
		Query.Returns(Some<Beer>.Records)
			.Then(Some<Glass>.Records)
			.Then(Some<Napkin>.Records);

Now, `results.Set3` contains napkins.

# One-to-Many Mappings #

Sometimes when you return multiple recordsets, what you want to do is return a tree of objects in a one-to-many relationship. Let's say there is more than one appropriate glass for a given kind of beer. Each beer (one) can have many glasses.

	class Beer
	{
		public int ID;
		public ICollection<Glass> Glass; // multiple proper glass per beer
	}

You would probably return recordsets like this:

	SELECT * FROM Beer WHERE Type="IPA"
	SELECT b.ID, g.* FROM Glasses g JOIN RightGlass(...) ...

NOTE: when reading a set of child records, Insight will always assume the first column is the parent ID.

Now, we want to roll up the glasses into the Glass collection on Beer:

	List<Beer> yum = c.QuerySql<Beer>(sql, parameters,
		Query.Returns(Some<Beer>.Records)
			.ThenChildren(Some<Glass>.Records);

Here, we're telling Insight to expect two recordsets. The first one contains Beer. The second one is expected to be a set of Glasses. Insight will create a list of them and attempt to automatically add them to an `ICollection<Glass>` on Beer.

The field/property can be any of the following types:

* IList<T>
* IEnumerable<T>
* ICollection<T>
* List<T>

Insight will assign a List<T> to the property.

## How Insight will handle the one-to-many mapping ##

To determine the ID field, Insight uses the following order of precedence:

1. The field explicitly requested in the structure definition.
2. The field on the class marked with the `[RecordId]` attribute.
3. A field called "ID".
4. A field called "classID", where "class" is the short name of the class.
5. A field ending in "ID".

If Insight doesn't pick the right ID field, you can specify it:

		Query.Returns(Some<Beer>.Records)
			.ThenChildren(Some<Glass>.Records, b => b.MyIdentifier);

Or to do it in one place, add the `RecordId` attribute on the ID property. It only needs a getter.

	public class Beer
	{
		[RecordId]
		public int MyIdentifier;
		...
	}

To determine the target list field, Insight uses the following order of precedence:

1. The field explicitly requested in the structure definition.
2. The field on the class marked with the `[ChildRecords]` attribute that matches the type of the list.
3. Any field matching the type of the list.

If Insight doesn't pick the right target field, you can specify it:

		Query.Returns(Some<Beer>.Records)
			.ThenChildren(Some<Glass>.Records, into: (b, g) => b.BarShelf = g.ToList);


Other Notes:

* The way it's coded, it's possible for more than one parent to have a given ID, so it's actually possible to return many-to-many mappings. Woot. 

# Multiple One-to-Many Mappings #

You can also return more than two recordsets. In this case Napkins go into Beer too:

	Query.Returns(Some<Beer>.Records)
		.ThenChildren(Some<Glass>.Records)
		.ThenChildren(Some<Napkin>.Records);

Subsequent recordsets can have their own one-to-one relationships:

	Query.Returns(Some<Beer>.Records)
		.ThenChildren(OneToOne<Glass, Napkin>.Records);

You can also create multiple levels. Here we put the napkins under the glasses instead of the beer:

	Query.Returns(Some<Beer>.Records)
		.ThenChildren(Some<Glass>.Records)
		.ThenChildren(Some<Napkin>.Records, parents: beer => beer.Glasses);

Note that if Glass has a funky ID field, or a weird list, you would specify it this way: 

	Query.Returns(Some<Beer>.Records)
		.ThenChildren(Some<Glass>.Records)
		.ThenChildren(Some<Napkin>.Records, 
			parents: beer => beer.Glasses,
			id: glass => glass.StemNumber,
			into: (glass, napkins) => glass.NapkinRing = napkins);

# Mix and Match Your Mappings #

Okay, so this is getting a little crazy, but it's not icky at all! Returning 4 recordsets, with two parents and two children!

	Results<Beer, Wine> results = c.QuerySql(
		@"	SELECT * FROM Beer WHERE Type="IPA"
			SELECT b.ID, g.* FROM Glasses g JOIN RightGlass(...) ...
			SELECT * FROM Wine WHERE Type="IPA"
			SELECT w.ID, g.* FROM Glasses g JOIN RightGlass(...) ...",
			parameters,
 	Query.Returns(Some<Beer>.Records)
		.ThenChildren(Some<Glass>.Records)
		.Then(Some<Wine>.Records);
		.ThenChildren(Some<Glass>.Records)

Beer and Wine are the parents. Both of them have glasses as children.

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
		[Recordset(0, typeof(Beer), typeof(Glass))]
		IList<Beer> GetBeerByType(string type);
	}

We want to be able to do the same recordset mapping for interfaces. Multiple results is pretty easy:

	interface IBarDatabase
	{
		[Sql("SELECT * FROM Beer; SELECT * FROM Glass")]
		Results<Beer, Glass> GetAllBeerAndGlasses();
	}

(Oh yeah, here we did inline SQL.)

Note how Insight detects that you have two recordsets because you declared the return type to be `Results<Beer, Glass>`. Unfortunately, that doesn't work for arbitrary trees, so we're going to have to use attributes.

By adding multiple `Recordset` attributes, we can tell Insight what to expect.

Let's try multiple recordsets:

	interface IBarDatabase
	{
		[Sql("SELECT * FROM Beer JOIN Glass; SELECT * from Wine JOIN Glass...;")]
		[Recordset(0, typeof(Beer), typeof(Glass))]
		[Recordset(1, typeof(Wine), typeof(Glass))]
		Results<Beer, Wine> GetAllBeerAndGlasses();
	}

**Notice how the recordset attributes need to be numbered.**
**QUESTION: 0 based or 1 based?**

Let's try a one-to-many relationship:

	interface IBarDatabase
	{
		[Sql("SELECT * FROM Beer; SELECT b.ID, g.* FROM Glasses g JOIN RightGlass(...) ...")]
		[Recordset(1, typeof(Glass))]
		IList<Beer> GetAllBeerAndGlasses();
	}

Here, we didn't have to specify the types for Recordset 0. Insight knows that it contains Beer. (It can always find Beer...). So if you're not adding OneToOne mappings or children, you can skip the recordset.

If you needed to override the `id` and `into` properties:

	interface IBarDatabase
	{
		[Sql("SELECT * FROM Beer; SELECT b.ID, g.* FROM Glasses g JOIN RightGlass(...) ...")]
		[Recordset(0, typeof(Beer), Id="MyIdentifier")]
		[Recordset(1, typeof(Glass), Into="BarShelf")]
		IList<Beer> GetAllBeerAndGlasses();
	}

Three levels would be:
 
	interface IBarDatabase
	{
		[Sql("SELECT * FROM Beer; SELECT b.ID, g.* FROM Glasses g JOIN RightGlass(...) ...")]
		[Recordset(0, typeof(Beer))]
		[Recordset(1, typeof(Glass))]
		[Recordset(2, typeof(Napkin), Into="Glasses.Napkin")]
		IList<Beer> GetAllBeerAndGlassesAndNapkins();
	}

Note that here we told Insight to put the Napkins inside the Glasses.

**4.0-alpha1 - more than two child levels on an interface isn't implemented yet**

Mixing up one-to-many and one-to-many (Beer can go in many glasses, Wine snobs only use one particular glass):

	interface IBarDatabase
	{
		[Recordset(0, typeof(Beer))]
		[Recordset(1, typeof(Glass))]
		[Recordset(2, typeof(Wine), typeof(Glass))]
		Results<Beer, Wine> GetBeerAndWine();
	}

Mixing up unrelated and one-to-many:

	interface IBarDatabase
	{
		[Sql("SELECT * FROM Beer; SELECT b.ID, g.* FROM Glasses g JOIN RightGlass(...) ...")]
		[Recordset(0, typeof(Beer))]
		[Recordset(1, typeof(Glass), IsChild=true)]
		[Recordset(2, typeof(Wine))]
		[Recordset(3, typeof(Glass), IsChild=true)]
		Results<Beer, Wine> GetBeerAndWine();
	}

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

**4.0-alpha1 - fluent config not implemented yet.**
