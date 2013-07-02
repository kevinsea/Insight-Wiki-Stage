# PostgreSQL Provider #

The PostgreSQL Provider allows Insight.Database to work with your PostgreSQL database.

## Initializing the PostgreSQLInsightDbProvider ##

Before you can use Insight.Database with PostgreSQL, you must first install and register the provider in your application. The provider allows Insight to handle some of the connection-specific database issues.

1. Install the Insight.Database.Providers.PostgreSQL package from NuGet.
2. Call PostgreSQLInsightDbProvider.RegisterProvider().

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

## Known Issues ##

* Table-Valued Parameters are not currently supported by the Npgsql driver.
