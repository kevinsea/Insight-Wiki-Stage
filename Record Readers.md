When Insight goes to translate a record into an object, it uses the `IRecordReader` class.

There are several implementations:

* `Some<T>.Records` - uses auto-mapping to read some T from the record.
* `OneToOne<T1, T2>.Records` - uses auto-mapping to split the record into T1 and T2 objects, then automatically assembles them.

## How Auto-Mapping Works ##

By default, when you specify a One-To-One mapping, Insight automatically breaks up the recordset into objects, then assembles them for you.

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

When you do any of the following, you get the default automapping:

1. Adding the `Recordset` Attribute to your class.
2. In the generic parameters of the Query or ToList methods: `Query<Type1, Type2, Type3..>`.
3. Passing a OneToOne object to Query.

	[Recordset(typeof(Serving), typeof(Beer))]
	class Serving
	...

	var list = connection.Query<Serving, Beer>(...

	var list = connection.Query(...
		Query.Returns(OneToOne<Serving, Beer>);

## Splitting up the Columns ##

Insight will split up the result set as follows:

1. Go through each column from left to right (or 0 to n).
1. If the column maps to a property of the first class (Serving), then map it.
1. If the column does not map to the first class, and the column maps to the second class, then the second class must be starting.
1. Repeat until the columns are exhausted.

Note that you can repeat types if you return multiple objects of a given type. So `<Serving, Beer, Beer, Glass>` might be acceptable for a half-and-half.

## Customizing Column Division ##

If you don't like the way Insight divides up the columns, you can override it. Create your own instance of `OneToOne<>`, and specify the idColumns:

	var mapping = new OneToOne<Serving, Beer>(idColumns: new Dictionary<Type, string<()
	{
		{ typeof(Serving), "ServingID", }
		{ typeof(Beer), "BeerID" }
	});

Then pass your own mapping into Query.Returns:

	var list = connection.Query(...
		Query.Returns(mapping);

## Assembling the Object Graph ##

Now we have a set of objects for the row. In this case a Serving, a Beer, and a Glass. Insight needs to put the objects together into a hierarchy. 

1. The first class is always the root of the hierarchy. In this case, a Serving.
1. The next object is a Beer. Serving has a property of type Beer, so it gets mapped to Serving.Beer.
1. The next object is a Glass. Serving has a property of type Glass, so it get mapped to Serving.Glass.
1. If Serving didn't have a property of type Glass, then Insight will look for a Glass-typed property on Beer. This allows for multiple levels of hierarchy.

## Customizing the Assembly of the Object Graph ##

If you don't like the way Insight assembles the object, you can teach it:

	var mapping = new OneToOne<Serving, Beer>(callback: (s, b) => s.Beer = b);

AsEnumerable<T> also accepts a callback Action. The Action gets one parameter per deserialized type. This is your opportunity to wire up the object references yourself.

	var reader = connection.GetReader(....);
	List<Serving> servings = reader.AsEnumerable<Serving, Beer, Glass>(
		callback: (Serving s, Beer b, Glass g) =>
		{
			s.Beer = b;
			s.Glass = g;
		});

The callback is an Action, not a Function. The returned object is always the first object. You just have a chance to resolve the references in the method. Note that any objects not linked to the main object will be thrown away.

## Reading Different Classes Per Record ##

Let's say you have a switch field and you want to return a different class depending on the field. You can use the `MultiReader` class to tell Insight how to deserialize individual records.

	var mr = new MultiReader<MyClass>(
		reader =>
		{
			switch ((string)reader["Type"])
			{
				default:
				case "a":
					return OneToOne<MyClassA>.Records;
				case "b":
					return OneToOne<MyClassB>.Records;
			}
		});

	var results = Connection().QuerySql(
		"SELECT [Type]='a', A=1, B=NULL UNION SELECT [Type]='b', A=NULL, B=2", Parameters.Empty, Query.Returns(mr));

## PostProcessing Records ##

Similarly, you can use the `PostProcessRecordReader` to make additional changes to each record as its read.
 
		[Test]
		public void PostProcessCanReadFieldsInAnyOrder()
		{
			var pr = new PostProcessRecordReader<MyClassA>(
				(reader, a) =>
				{
					// after A is read, look at the reader and make more changes
					if (reader["Type"].ToString() == "a")
						a.A = 9;
					return a;
				});

			var results = Connection().QuerySql("SELECT [Type]='a', A=1, B=NULL", Parameters.Empty, Query.Returns(pr));

			Assert.AreEqual(1, results.Count);
			Assert.IsTrue(results[0] is MyClassA);
			Assert.AreEqual(9, ((MyClassA)results[0]).A);
		}

## Custom IRecordReaders ##

All of this is implemented through record readers. If this flexibility isn't enough for you, you can create your own implementation of `IRecordReader` to read data from the stream. You can pass it in anywhere Insight takes a `Some<T>` or `OneToOne<T...>`.

The most common use for this is when you don't have a default constructor for your class. Unfortunately, if you don't have a default constructor, Insight can't create your object for you, so you'll have to do the reads manually.

The easiest way to do this is to use the `CustomRecordReader<T>` class:

	// create a reader that manually reads the record
	private static CustomRecordReader<TestChild> _reader = new CustomRecordReader<TestChild>(
		r => new TestChild((int)r["ChildA"], (int)r["ChildB"])
	);

	// pass the reader into any of the query definition methods
	var results = Connection().QuerySql("SELECT ID=1; SELECT ParentID=1, ChildA=2, ChildB=3", null,
		Query.Returns(Some<TestParent>.Records)
			.ThenChildren(reader);

You can also use an inline reader if you want:

	// pass the reader into any of the query definition methods
	var results = Connection().QuerySql("SELECT ID=1; SELECT ParentID=1, ChildA=2, ChildB=3", null,
		Query.Returns(Some<TestParent>.Records)
			.ThenChildren(CustomRecordReader<TestChild>.Read(r => new TestChild((int)r["ChildA"], (int)r["ChildB"]));

NOTE: Insight turns on sequential reads, so you have to read the fields in order or use the `CachedDbDataReader` class.

[[Specifying Result Structures]] - BACK || NEXT- [[Query Readers]]