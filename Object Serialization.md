Sometimes you want a sub-object to be serialized into a string column. Like this:

	public class GiftPack
	{
		public int ID;

		public Glass Glass;
	}

	public class Glass
	{
		public int Size;
		public string Style;
	}

	CREATE TABLE GiftPack
	AS
		ID int IDENTITY,
		Style [varchar](MAX)
	GO

Here, we want to put the class into the Style column and we want it serialized as a string. Now, if the column is an XML column, Insight will automatically use the DataContractSerializer to pack up your object. Here we have a string, so Insight can't do it automatically. We can give it a clue with a column attribute:

	public class GiftPack
	{
		public int ID;

		[Column(SerializationMode=SerializationMode.Json)]
		public Glass Glass;
	}

By adding the Column attribute, we tell Insight to convert the GiftPack to JSON when storing it, and to convert it back when reading it from a recordset. There are several built-in serialization modes:

* Default - see ToString
* Xml - the DataContractSerializer is used to convert to/from XML
* Json - the DataContractJsonSerializer is used to convert to/from XML
* ToString - the ToString method is used to convert the object to a string. An error occurs when reading the object.
* Custom - see Custom Serializers below.

Note that object serialization only occurs when a field is matched to a string column.

## Using Mapping Rules to Control Serialization ##

If you don't like adding attributes to your objects, you can also set up mapping rules to control the serialization formats:

	// specify serialization by type and field name
	DbSerializationRule.Serialize<GiftPack>("Glass", SerializationMode.Xml);

	// specify serialization by type and field type (applies to all fields of the type)
	DbSerializationRule.Serialize<GiftPack>(typeof(Glass), SerializationMode.Xml);

	// specify serialization by field type (applies across all types/tables) 
	DbSerializationRule.Serialize<Glass>(SerializationMode.Xml);

## Using JSON.NET for JSON Serialization ###

If you think DataContractJsonSerializer is slow, and you would rather use Newtonsoft's JSON.NET, you can:

1. Install the Insight.Database.Json package.
2. In your startup code, call `Insight.Database.Json.JsonNetObjectSerializer.Initialize()`

Then Insight will use the JSON.NET serializer for JSON conversions.

## Boolean Serializers ##

Since a lot of databases don't have a bit type (e.g. Oracle), you might need to convert boolean values to things like "T" and "F". Starting in v5.2, you get a few built-in boolean serializers:

* BooleanYNSerializer
* BooleanTFSerializer
* BooleanTrueFalseSerializer
* Boolean10Serializer
* BooleanSerializer - roll your own like new `BooleanSerializer(DbType.String, "Foo", "Bar")` 

You can install them with any of the methods, including globally like this:

	DbSerializationRule.Serialize<bool>(new BooleanYNSerializer());

Or for a specific field:

	DbSerializationRule.Serialize<MyClass>("MyField", new BooleanYNSerializer());

Or with an attribute:

	public class MyClass
	{
		[Column(SerializationMode=SerializationMode.Custom, Serializer=typeof(Boolean10Serializer))]
		public bool MyField;
	}

## Custom Serializers ##

If you have a need for custom serialization, you can create your own object serializer:

	public class MyDbObjectSerializer : DbObjectSerializer
	{
		public override object SerializeObject(Type type, object o)
		{
			return object.ConvertToString();
		}

		public override object DeserializeObject(Type type, object encoded)
		{
			return encoded.ConvertToObject();
		}
	}

It's also possible to do a binary serializer by overriding additional methods.

Now, you can just add a Column attribute to your field: 

	public class GiftPack
	{
		public int ID;

		[Column(SerializationMode=SerializationMode.Custom, Serializer=typeof(MyDbObjectSerializer))]
		public Glass Glass;
	}

## Custom Serializer Rules ##

If you want to override the way Insight picks serializers:

	public class MyDbSerializationRule : IDbSerializationRule
	{
		public IDbObjectSerializer GetSerializer(Type recordType, Type memberType, string memberName)
		{
			return MyDbObjectSerializer;
		}
	}

	DbSerializationRule.AddRule(new MyDbSerializationRule());