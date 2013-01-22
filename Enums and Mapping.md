# Enums and Mapping #

Insight handles most conversions between enums and other data types.

## Enums as Parameters ##

Enums are always sent to the database as an **integer** representation.

	var results = connection.QuerySql<string>("SELECT @p", new { p = MyEnum.One });
	Console.Write(results.First());
	// outputs "1"

If you want to store your enums as **strings**, you must convert them yourself.

	var results = connection.QuerySql<string>("SELECT @p", new { p = MyEnum.One.ToString() });
	Console.Write(results.First());
	// outputs "One"

Why? With stored procedures, we could tell that the database is expecting a string/varchar, but we don't have type information when executing SQL Text commands. We don't want the stored data to change depending on whether we are calling a procedure or SQL Text.

## Enums as Output Columns in a Class/Struct ##

Insight will coerce column data to an enum field/property. It will automatically convert integers by size, as well as converting strings to enums by using Enum.Parse. In short, these conversions will generally "just work".

	class MyClass
	{
		public MyEnum Foo;
	}

	var results = connection.QuerySql<MyClass>("SELECT Foo=CONVERT(tinyint, 1)");
	Console.Write(results.First.Foo);
	// outputs "One"

	var results = connection.QuerySql<MyClass>("SELECT Foo='One'");
	Console.Write(results.First.Foo);
	// outputs "One"

	var results = connection.QuerySql<MyClass>("SELECT Foo='1'");
	Console.Write(results.First.Foo);
	// outputs "One"

Why? This gives you the most flexibility when querying multiple tables, etc. and we always know what you want to convert **to**, so we do our best to make it happen.

## Enums as Single-Column Outputs ##

When returning enums as single-column outputs, the underlying type of the enum must match the return type of the command. Insight will NOT perform type coersions on data returned.

	// succeeds!
	var results = connection.QuerySql<MyEnum>("SELECT CONVERT(int, 1)");

	// fails
	var results = connection.QuerySql<MyEnum>("SELECT CONVERT(bigint, 1)");
	var results = connection.QuerySql<MyEnum>("SELECT 'One'");

Why? In order to do advanced conversions, we would need to do extra analysis on the result stream. The extra overhead doesn't justify the number of times you would use this.

If you need to do advanced conversions on your single-column enums, consider wrapping the enum in a struct and using the struct deserializer that supports the conversions:

	struct Wrapper<T> { public T Value; }
	var results = connection.QuerySql<Wrapper<MyEnum>>("SELECT Value=CONVERT(bigint, 1)");

