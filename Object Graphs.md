# Object Graphs #

The mapping engine for a result set supports object graphs. Let's say you kept track of the beer you poured in another table. When you return Servings, you also return the Beer that was poured and the Glass that it went in.

	CREATE TABLE Servings
	(
		[When] [datetime]
		[BeerName] [varchar](32),
		[GlassName] [varchar](32)
	)

	CREATE PROCEDURE GetServings
	AS
		SELECT [When], b.*, g.*
			FROM Servings s
			JOIN Beer b ON (s.BeerName = b.Name)
			JOIN Glasses g ON (s.GlassName = g.Name)

	class Serving
	{
		public DateTime When;
		public Beer Beer;
		public Glass Glass;
	}

You can tell Insight to to deserialize these three types from the result set if the columns are returned properly.

* The columns should be grouped by type.
* The types should be in the same order as the columns.
* NOTE: not all of the columns need to be returned for any of the types.

## Specifying Object Graphs ##

You can tell Insight the classes to expect in a recordset in a few ways:

1. By adding the `DefaultGraph` Attribute to your class.
2. In the generic parameters of the Query or ToList methods: `Query<Type1, Type2, Type3..>`.
3. By passing a type to the `withGraph` or `withGraphs` parameters to a method.

## The DefaultGraph Attribute ##

If your result sets are consistent, you can tell Insight to always expect the classes to be in the same order by adding the DefaultGraph Attribute to your class.

The following will tell Insight that when Servings are deserialized, expect Beer and Glasses to follow, in that order.

	[DefaultGraph (typeof (Graph<Servings, Beer, Glasses>))]
	class Servings
	{
		...
	}

Insight is smart enough to detect when Beer or Glasses are returned in a recordset. If they aren't returned, the sub-objects are not created.

The DefaultGraph can also be overridden by one of the other two methods.

## Specifying Object Graphs in Generic Parameters ##

You can deserialize the results of GetServings by specifying a few more class arguments to the Query method:

	var servings = Database.Connection().Query<Serving, Beer, Glass>("GetServings", Parameters.Empty);
	foreach (var serving in servings)
		Console.WriteLine("{0} {1}", serving.Beer.Name, serving.Glass.Ounces);

## Specifying Object Graphs with the withGraph or withGraphs Parameters ##

Many methods accept a withGraph or withGraphs parameter. This allows you to override the DefaultGraph for specific queries:

	var servings = Database.Connection().Query<Serving>("GetServings", withGraph: typeof(Graph<Serving, Beer, Glass>);

This is useful when programmatically setting the return types or when using [[Dynamic Database Calls]].

## Splitting up the Columns ##

Insight will split up the result set as follows:

1. Go through each column from left to right (or 0 to n).
1. If the column maps to a property of the first class (Serving), then map it.
1. If the column does not map to the first class, and the column maps to the second class, then the second class must be starting.
1. Repeat until the columns are exhausted.

Note that you can repeat types if you return multiple objects of a given type. So `<Serving, Beer, Beer, Glass>` might be acceptable for a half-and-half.

## Customizing Column Division ##

AsEnumerable<T> accepts an idColumns parameter to control the mapping from columns to objects. This is a map from Type to string. It tells Insight the names of the columns that determine the break point between classes. This is useful if you are returning fields with the same names but in different classes.

## Assembling the Object Graph ##

Now we have a set of objects for the row. In this case a Serving, a Beer, and a Glass. Insight needs to put the objects together into a hierarchy. 

1. The first class is always the root of the hierarchy. In this case, a Serving.
1. The next object is a Beer. Serving has a property of type Beer, so it gets mapped to Serving.Beer.
1. The next object is a Glass. Serving has a property of type Glass, so it get mapped to Serving.Glass.
1. If Serving didn't have a property of type Glass, then Insight will look for a Glass-typed property on Beer. This allows for multiple levels of hierarchy.

## Customizing the Assembly of the Object Graph ##

AsEnumerable<T> also accepts a callback Action. The Action gets one parameter per deserialized type. This is your opportunity to wire up the object references yourself.

	var reader = connection.GetReader(....);
	List<Serving> servings = reader.AsEnumerable<Serving, Beer, Glass>(
		callback: (Serving s, Beer b, Glass g) =>
		{
			s.Beer = b;
			s.Glass = g;
		});

The callback is an Action, not a Function. The returned object is always the first object. You just have a chance to resolve the references in the method. Note that any objects not linked to the main object will be thrown away.

