# Object Hierarchies #

The mapping engine for a result set supports object hierarchies. Let's say you kept track of the beer you poured in another table:

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

You can deserialize the results of GetServings by specifying a few more class arguments to the Query method:

	class Serving
	{
		public DateTime When;
		public Beer Beer;
		public Glass Glass;
	}

	var servings = Database.Connection().Query<Serving, Beer, Glass>("GetServings", Parameters.Empty);
	foreach (var serving in servings)
		Console.WriteLine("{0} {1}", serving.Beer.Name, serving.Glass.Ounces);

By specifying `<Serving, Beer, Glass>`, Insight knows that you are trying to deserialize these three types from the result set.

* The columns should be grouped by type.
* The types should be in the same order as the columns.

Insight will split up the result set as follows:

1. Go through each column from left to right (or 0 to n).
1. If the column maps to a property of the first class (Serving), then map it.
1. If the column does not map to the first class, then the second class must be starting, so start looking in the second class.
1. Repeat until the columns are exhausted.

Note that you can repeat types if you return multiple objects of a given type. So `<Serving, Beer, Beer, Glass>` might be acceptable for a half-and-half.

Now we have a set of objects for the row. In this case a Serving, a Beer, and a Glass. Insight needs to put the objects together into a hierarchy. 

1. The first class is always the root of the hierarchy. In this case, a Serving.
1. The next object is a Beer. Serving has a property of type Beer, so it gets mapped to Serving.Beer.
1. The next object is a Glass. Serving has a property of type Glass, so it get mapped to Serving.Glass.
1. If Serving didn't have a property of type Glass, then Insight will look for a Glass-typed property on Beer. This allows for multiple levels of hierarchy.

## Customizing the Deserialization ##
If this doesn't work for you, there are two hooks for controlling deserialization. Both of these are on the AsEnumerable class.

AsEnumerable<T> accepts an idColumns parameter to control the mapping from columns to objects. This is a map from Type to string. It tells Insight the names of the columns that determine the break point between classes. This is useful if you are returning fields with the same names but in different classes.

AsEnumerable<T> also accepts a callback Action. The Action gets one parameter per deserialized type. This is your opportunity to wire up the object references yourself. It would go something like this (no, I didn't test this):

	var reader = connection.GetReader(....);
	List<Serving> servings = reader.AsEnumerable<Serving, Beer, Glass>(
		callback: (Serving s, Beer b, Glass g) =>
		{
			s.Beer = b;
			s.Glass = g;
		});

The callback is an Action, not a Function. The returned object is always the first object. You just have a chance to resolve the references in the method. Note that any objects not linked to the main object will be thrown away.

