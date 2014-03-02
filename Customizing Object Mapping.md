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

## Setting Up Mapping Rules ##

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
		// add a custom mapping handler
		ColumnMapping.Tables.AddHandler(new MyCustomMappingHandler());
	}

	class MyCustomMappingHandler : ICustomMappingHandler
	{
		void ICustomMappingHandler.HandleColumnMapping(object sender, ColumnMappingEventArgs e)
		{
			e.TargetFieldName = "_" + e.TargetFieldName;
		}
	}

The ColumnMappingEventArgs provides information about the mapping operation and allows you to customize the mapping.

* TargetFieldName - set this to the name of the field that you want to map to. When you receive it, it will contain the result of any previous mappings that have been applied.
* Cancelled - set this to true to skip the given column.
* CommandText - the current command text that is being mapped. This will be null when mapping tables.
* CommandType - the current Command Type that is being mapped. This will be null when mapping tables.
* FieldIndex - the index into the reader's columns or the parameter list.
* Reader - The IDataReader that is currently being mapped. You can use this to query the result set for more information about the current mapping operation. For example, if the IDataReader is a SqlDataReader, you can use GetSchemaTable to get information about the underlying table that was queried. This will be null when mapping parameters.
* Parameters - the list of parameters that is currently being mapped.
* TargetType - the type of the object that is currently being mapped.

## Column Overrides ##

If you need to override columns for individual queries, the best way to do it is to create a new record reader:

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


[[Specifying Result Structures]] - BACK || NEXT- [[Custom Result Objects]]