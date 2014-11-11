Sometimes you have an ugly database that you aren't allowed to modify. It may be ugly, AND you aren't allowed to create new views or stored procedures to make it look nice. But you want your code to look nice, right?

You have six choices:

1. Make a class, bind the data to private members and expose pretty properties.
1. Use ColumnAttribute on your classes to override the mapping.
1. Set up some mapping rules.
1. Install a custom mapping handler.
1. Add an override to a record reader.
1. Use [[FastExpando and Mutations]].

For the example below, let's assume we are running the following SQL Query and we want property names without prefixes:

	SELECT intID, vcName FROM Beer

## Using Private Members ##

You can create your classes with private members that Insight will bind to. This works, but it's not the cleanest.

	class Beer
	{
		public int ID { get { return intID; } set { intID = value; }
		private int intID;

		public string Name { get { return vcName; } set { vcName = value; }
		private string vcName;
	}

## ColumnAttribute ##

You can also tell Insight the column name for a property by using the ColumnAttribute. This is handy if you have different rules for different tables.

	class Beer
	{
		[Column("intID")]
		public int ID { get; set; }

		[Column("vcName")]
		public string Name { get; set; }
	}

## Setting Up Transformation Rules ##

If your databases use some form of a naming convention, you can add some mapping rules. Insight exposes the ColumnMapping class to let you add rules to table mappings or parameter mappings in your startup code.

You can add mapping rules like removing prefixes, suffixes or regex matches, or add replacement regex rules.

	public void OnApplicationStartup()
	{
		// remove foo from the start of column names
		// remove bar from the end of column names
		// remove int or vc from the start of column names
		// correct a misspelling of Beer
		ColumnMapping.Tables.RemovePrefixes("foo")
			.RemoveSuffixes("bar")
			.RemoveRegex("^((int)|(vc))")
			.ReplaceRegex("Beeer", "Beer");
	}

If you want to restrict a rule to a specific type (and its subtypes), use the generic version of the method:

	public void OnApplicationStartup()
	{
		// correct a misspelling of Beer, but only on Beer and subtypes.
		ColumnMapping.Tables
			.ReplaceRegex<Beer>("Beeer", "Beer");
	}

There are separate rules for column mappings and parameter mappings:

* ColumnMapping.Tables - handles mapping result sets to objects, objects to Table-Valued Parameters, and bulk copy mappings.
* ColumnMapping.Parameters - handles mapping objects to stored procedure parameters.

The built-in rules are:
* RemovePrefixes
* RemoveSuffixes
* RemoveStrings
* RemoveRegex
* ReplaceRegex

Notes:

* Mappings are always case-insensitive.
* Rules are evaluated in the order they are added, and are cumulative.

## Custom Mapping Handlers ##

If these rules aren't enough for you, you can install a custom mapping handler. The first time Insight encounters a mapping, it generates an event that you can handle to do the custom mapping.

	public void OnApplicationStartup()
	{
		// add custom mappers
		ColumnMapping.Tables.AddMapper(new MyColumnMapper());
		ColumnMapping.Parameters.AddMapper(new MyParameterMapper());
		ColumnMapping.All.AddTransform(new MyMappingTransform();
	}

	class MyMappingTransform : IMappingTransform
	{
		string TransformDatabaseName(Type type, string databaseName)
		{
			// someone always gets this wrong... 
			return databaseName.Replace("there", "their");
		}
	}

	class MyColumnMapper : IColumnMapper
	{
		string MapColumn(Type type, IDataReader reader, int column)
		{
			if (type != typeof(Beer))
				return null;			// null allows another handler to try

			if (reader.GetName(column) == "Glass")
				return "Stein";			// a string value specifies the name of the field/property

			return "__unmapped__";		// returning an invalid string value prevents other handlers from mapping 
		}
	}

	class MyParameterMapper : IParameterMapper
	{
		string MapParameter(Type type, IDbCommand command, IDataParameter parameter)
		{
			if (command.CommandText != "MyProc")
				return null;			// null allows another handler to try

			if (parameter.ParameterName == "Glass")
				return "Stein";			// a string value specifies the name of the field/property

			return "__unmapped__";		// returning an invalid string value prevents other handlers from mapping 
		}
	}

Notes on mapping:

* The mapping methods are called once when the binding code is generated.
* Transforms are applied before mapping occurs.
* All transforms are run in sequence, so you can chain multiple transforms.
* The mappers are called in order until one of them returns a non-null value.
* If the field is not found on the given type, the column/parameter is unbound.
* For parameters and TVPs, you can specify subobjects by mapping to "field.childfield". Insight will split on the dots and drill into child objects.

## Column Overrides ##

If you need to override columns for individual queries, the easiest way to do it is to create a new record reader:

	// the structure object is immutable, so you should cache it
	var structure = new OneToOne<MyObject>(
		// if you don't override a column, the default rules are used 
		new ColumnOverride<MyObject>("Foo", "ID")
	);

	var result = Connection().QuerySql(
		"SELECT Foo=1, ID=2",
		null,
		Query.Returns(structure));

By default, the `OneToOne` class does an automatic mapping from the recordset to your object. Here, we created a new `OneToOne` that reads `MyObject`, but puts the `Foo` column into the `ID` property. Then we can pass that into any method that has a `returns` parameter. You can also use it to build multiple results, parent/child, etc.

## Binding Into Children ##

Sometimes you want Insight to be able to automatically peek into an object to extract parameters. You can tag classes with the `BindChildrenFor` attribute, and Insight can then look inside for you.

	[BindChildrenFor(BindFor.All)]
	class Beer
	{
		public int ID;
		public Glass Glass;
	}

	class Glass
	{
		public int GlassID;
	}

	CREATE PROC MyProc(@ID int, @GlassID int)

	// GlassID is not found on Beer, so it looks in Glass
	Connection().Execute("MyProc", beer);

If you don't want to do this with attributes, you can enable it through configuration:

	ColumnMapping.All.EnableChildBinding<Beer>();	// insight can peek into beer anytime
	ColumnMapping.Parameters.EnableChildBinding<Glass>(); // peek into glasses when parameter binding
	ColumnMapping.All.EnableChildBinding<object>();	// always use this handy feature

Unfortunately, this has to be done at the class level, and can't be turned on/off at the query/statement level at this time without using a custom mapper.

## Constructors ##

Insight will use the following rules for selecting a constructor:

1. If a constructor is marked with `[SqlConstructor]`, then Insight will use that constructor.
2. Otherwise, if there is *exactly* one constructor, then Insight will use that.
3. Otherwise, Insight will use the default constructor. If there is no default constructor, an exception is thrown.

When mapping each constructor parameter:

* Insight will assume that the parameter name matches a field/property.
* Constructor parameters will inherit any column mappings and custom serializers of the field/property with the same name.
* Finally, after the constructor is called, Insight will map any remaining fields/properties that were not passed to the constructor.

Example:

       public class ConstructorTest
       {
            // this column mapping is also applied to the parameter
            // also note that this has a readonly field
            [Column("foo")]
            public readonly int A;

			public int B;

            // also, the name 'a' needs to match the field/property A (case-insensitive)
            public ConstructorTest(int a)
            {
                A = a;
            }
        }

        [Test]
        public void TestConstructorTest()
        {
            var a = Connection().QuerySql<ConstructorTest>("SELECT Foo=3, B=4").First();
            Assert.AreEqual(3, a.A);
            Assert.AreEqual(4, a.B);
        }

       public class TwoConstructors
       {
            public int A;
			public int B;

			public TwoConstructors()
			{
			}

			// tell Insight that this is the constructor you want to use
			[SqlConstructor]
            public TwoConstructors(int a, int b)
            {
                if (a == b) throw new ArgumentException("must not equal");
            }
        }

 

[[Specifying Result Structures]] - BACK || NEXT- [[Custom Result Objects]]