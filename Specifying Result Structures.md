Now that you know how each individual record is read, it's time to look at the different ways to read the results of a query.

## One Object ##

Reading a single object:

	Beer beer = connection.Single<Beer>("GetABeer");

## List of Objects ##

Reading a list of objects:

	IList<Beer> beer = connection.Query<Beer>("GetAListOfeer");

## Two Lists of Objects ##

Reading two lists of objects:

	var results = connection.QueryResults<Beer, Glass>("GetAllBeersAndAllGlasses");
	IList<Beer> beers = results.Set1;
	IList<Glass> glasses = results.Set2;

You can return up to 16 sets of objects this way.

This is equivalent to:

	connection.Query("GetABeerAndProperGlass", Parameters.Empty,
		Query.Returns(Some<Beer>.Records)
			.Then(Some<Glass>.Records);

Keep an eye on Query.Returns. It gets pretty powerful.

## One-To-One Relationships ##

Reading a list of objects with a one-to-one mapping:

	IList<Beer> beers = connection.Query<Beer, Glass>("GetABeerAndProperGlass");
	Beer beer = beers.First();
	Glass glass = beer.Glass;

This is equivalent to:

	connection.Query("GetABeerAndProperGlass", Parameters.Empty,
		Query.Returns(OneToOne<Beer, Glass>.Records);

Insight will automatically split each record into Beer objects and Glass objects, then assemble them properly.

If you almost always return two objects together, you can put a `RecordsetAttribute` on the class, so Insight will always try to read in the subobject. It's ok if the subobject isn't there.

	[Recordset(typeof(Beer), typeof(Glass))]
	class Beer { ... }


## One To Many Relationships ##

Here's where we start to get tricky. For anything advanced, we need to specify the structure that the query returns:

	CREATE PROC GetBeerAndPossibleGlasses AS
		SELECT * FROM Beer
		SELECT b.BeerID, g.8 FROM Beer JOIN Glasses ON (...)

	class Beer
	{
		public int ID;
		public IList<Glass> Glasses;
	}

	var results = connection.Query("GetBeerAndPossibleGlasses", Parameters.Empty,
		Query.Returns(Some<Beer>.Records)
			.ThenChildren(Some<Glasses>.Records);

Now Insight knows that you want a list of beer records, each with a list of glasses that match.

## Automatic Assembly ##

Insight will assume that the `Beer` class has an ID field, and that the first column in the glasses query is the Beer ID. Then it will break up the glasses into lists and assign them into your Beer objects.

To determine the ID field, Insight uses the following order of precedence:

1. The field explicitly requested in the structure definition.
2. The field on the class marked with the `[RecordId]` attribute.
3. A field called "ID".
4. A field called "classID", where "class" is the short name of the class.
5. A field ending in "ID".

To determine the target list field, Insight uses the following order of precedence:

1. The field explicitly requested in the structure definition.
2. The field on the class marked with the `[ChildRecords]` attribute that matches the type of the list.
3. Any field matching the type of the list.

The list field/property can be any of the following types:

* IList<T>
* IEnumerable<T>
* ICollection<T>
* List<T>

If Insight doesn't guess the ID field or list field right, you can tell it with attributes.

	class Beer
	{
		[RecordId] public int ID;
		[ChildRecords] public IList<Glass> Glasses;
	}

Or you can tell it with hints on the query:


	var results = connection.Query("GetBeerAndPossibleGlasses", Parameters.Empty,
		Query.Returns(Some<Beer>.Records)
			.ThenChildren(
				Some<Glasses>.Records,
				id: beer => beer.ID,
				into: (beer, glass) => beer.Glasses = glass);

Other Notes:

* The way it's coded, it's possible for more than one parent to have a given ID, so it's actually possible to return many-to-many mappings. Woot. 

## Mixing and Matching ##

You can mix and match `Then` and `ThenChildren` and also `OneToMany` records. Using these pieces, you can make almost any structure out of your data.

This is getting a little crazy, but it's not icky at all! Returning 4 recordsets, with two parents and two children!

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

## Three or more Levels ##

You can also handle hierarchies that have more than two levels. In this case, you'll have to use the `parents` hint on the ThenChildren method.

	class Order
	{
		public int OrderID { get; set; }
		public string Waitress { get; set; }
		public int TableNumber { get; set; }
		public List<Sampler> Samplers { get; set; }
	}
	class Sampler
	{
		public int SamplerID { get; set; }
		public string Name { get; set; }
		public List<Beer> Beers { get; set; }
	}
	class Beer
	{
		...
	}

This can be handled with three recordsets:

	SELECT Order.*
	SELECT OrderID, Sampler.*
	SELECT SamplerID, Beer.*

And now we tell Insight that the last two recordsets are children, and we tell it that the Beer goes into Samplers.
	
	var results = c.Query(proc, params, Query.Returns(Some<Order>.Records)
	    .ThenChildren(Some<Sampler>Records)
	    .ThenChildren(Some<Beer>.Records,
	                parents: o => o.Samplers,
	                id: s => s.SamplerID);

With this extension method, after reading all of the Orders and Samplers, Insight will gather up all of the Samplers and attempt to assign the Beers into the Samplers. 

[[Mapping Results to Objects]] - BACK || NEXT- [[Record Readers]]