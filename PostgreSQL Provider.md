# PostgreSQL Provider #

The PostgreSQL Provider allows Insight.Database to work with your PostgreSQL database.

## Initializing the PostgreSQLInsightDbProvider ##

Before you can use Insight.Database with PostgreSQL, you must first install and register the provider in your application. The provider allows Insight to handle some of the connection-specific database issues.

1. Install the Insight.Database.Providers.PostgreSQL package from NuGet.

## Returning Recordsets as Refcursors ##

When returning large recordsets, it can be more efficient to return a refcursor rather than a set of records. The driver will stream the results rather than pulling all of the records into memory.

The only issue with this is that the query must be run inside of a transaction in order for the cursors to stay open for the duration of the query.

				// NOTE: when returning cursors, you need to open the query in a transaction (eek)
				using (var connection = _connectionStringBuilder.OpenWithTransaction())
				{
					connection.ExecuteSql("CREATE TABLE PostgreSQLTestTable (p int)");
					connection.ExecuteSql(@"
						CREATE OR REPLACE FUNCTION PostgreSQLTestProc (i int) 
						RETURNS SETOF refcursor
						AS $$
						DECLARE
							rs refcursor;
						BEGIN 
							INSERT INTO PostgreSQLTestTable VALUES (@i);
							OPEN rs FOR SELECT * FROM PostgreSQLTestTable;
							RETURN NEXT rs;
						END;
						$$ LANGUAGE plpgsql;");
					var result = connection.Query<int>("PostgreSQLTestProc", new { i = 5 });
					Assert.AreEqual(1, result.Count);
					Assert.AreEqual(5, result[0]);
				}

## Output Parameters ##

Output Parameters are not supported in PostgreSQL. Instead, all output parameters are returned in the output recordset. You can use the Query<dynamic> extension method to get the output parameters.

				_connection.ExecuteSql(@"
					CREATE FUNCTION PostgreSQLTestOutput (x int, out z int)
					AS $$
					BEGIN 
						z := x; 
					END;
					$$ LANGUAGE plpgsql;");
				var result = _connection.Query("PostgreSQLTestOutput", new TestData() { X = 11, Z = 0 });

				dynamic output = result[0];

				Assert.AreEqual(11, output.Z);  

## Bulk Copy ##

Bulkcopy is supported. It is implemented by converting your objects to a CSV stream and sending them to the server. It's probably not the most efficient way to do it but it works.

## Support for JSON/JSONB Types ##

JSON fields work pretty well with PostgreSQL.

Let's say you have a table with a JSON data field:

	CREATE TABLE Users (Id integer NOT NULL, JsonData json)

You can automatically convert subobjects into/out of the JSON field, **as long as you set the serialization mode to JSON.**
 
	public class User
	{
		public long Id { get; set; }

		[Column(SerializationMode=SerializationMode.Json)]
		public TestData JsonData { get; set; }
	}

	var testData = new TestData() { X = 1, Z = 2 };
	var user = new User() { Id = 1, JsonData = input };

	// round-trip the object into JSON
	_connection.ExecuteSql("INSERT INTO Users (Id, JsonData) VALUES (@Id, @JsonData)", user);
	var result = _connection.QuerySql<User>(@"SELECT Users.* FROM Users").First();

This even works with stored procs.

	CREATE TYPE PostgreSQLTestType AS (Id integer, JsonData jsonb)
	CREATE OR REPLACE FUNCTION PostgreSQLTestExecute (Id integer, JsonData jsonb) 
	RETURNS SETOF PostgreSQLTestType
	AS $$
	BEGIN 
		RETURN QUERY SELECT id as id, JsonData as JsonData;
	END;
	$$ LANGUAGE plpgsql;

	var result = _connection.Query<User>("PostgreSQLTestExecute", user).First();

The only gotcha is if you want to post a **JSON string** into a JSON field using raw sql (not a stored proc). With raw SQL, there's no way for Insight to know that a particular parameter is JSON data. (We could make a special JsonString data type, but you really don't want us to do that.) 

The workaround is pretty simple: convert the string to JSON on the server side. See the example below, where we just add `::JSON` to the `@JsonData` parameter. You're already doing raw SQL, so this shouldn't be that bad.

	public class UserWithJsonString
	{
		public long Id { get; set; }
		public string JsonData { get; set; }	// you're serializing this manually? why, when we make it so easy for you!
	}
	
	var users = new UserWithJsonString()
	{
		Id = 1,
		JsonData = (string)JsonObjectSerializer.Serializer.SerializeObject(typeof (TestData), new TestData() { X = 1, Z = 2 })
	};
 
	_connection.ExecuteSql("INSERT INTO Users (Id, JsonData) VALUES (@Id, @JsonData::JSON)", users);


## Known Issues ##

* Table-Valued Parameters are not currently supported by the Npgsql driver.
