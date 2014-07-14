# Oracle Provider #

The only reason to make Insight work with Oracle is the challenge it provided. Fortunately, Insight can work almost perfectly with Oracle (including BulkCopy!). Just see the Known Issues below.

## ODP.NET ##

Insight.Database works with the [Oracle Data Provider for .NET (ODP.NET) (Download)](http://www.oracle.com/technetwork/topics/dotnet/index-085163.html).

## Initializing the OracleInsightDbProvider ##

Before you can use Insight.Database with Oracle, you must first install and register the provider in your application. The provider allows Insight to handle some of the wacky things needed to get ODP.NET to work.

1. Install the Insight.Database.Providers.Oracle package from NuGet.

## Returning Recordsets ##

If you are not familiar with Oracle PL/SQL, you will probably scratch your head trying to return a recordset from a procedure. You can't just SELECT BLAH like in SQL Server. You need to have a REF CURSOR parameter that you use to open your recordset:

	CREATE PROCEDURE GetData (
		rs1 OUT sys_refcursor,
		rs2 OUT sys_refcursor
	)
	IS
	BEGIN
		-- open two recordsets
		OPEN rs1 FOR SELECT 1 AS p from dual;
		OPEN rs2 FOR SELECT 'x' AS title from dual;
	END;

Insight will automatically take care of refcursor parameters for you. The recordsets are automatically read from the data reader and converted into objects.

(Also, you can't just do SELECT 1 AS p. You always have to select from a table. Fortunately, Oracle has a built in table called "dual" that always has one record.)

## Returning Multiple Recordsets from SQL Text ##

If you're not using stored procedures, you can still return multiple recordsets. If you use `OPEN :var FOR` Insight will automatically detect that you are returning refcursors and handle them appropriately. So the above query could be:

	var results = connection.QueryResultsSql<decimal, decimal>(@"
		BEGIN
			-- open two recordsets
			OPEN :rs1 FOR SELECT 1 AS p from dual;
			OPEN :rs2 FOR SELECT 'x' AS title from dual;
		END;
	");

Here, Insight detects that rs1 and rs2 are parameters. Since they are used in an OPEN...FOR statement, Insight will automatically bind output refcursors to them.

## Table-Valued Parameters ##

If you want to send lists of objects to the database, TVPs are the best way to do it. You will need a few pieces, and I apologize for not making it easier. Unfortunately, ODP.NET is hard-coded to look at attributes on classes in non-dynamic assemblies (or I would have just generated them for you).

NOTE: you only need this when sending TVPs TO the database. When you retrieve recordsets, Insight can automatically translate the recordsets to your objects.

In SQL, you will need a custom type and a custom table, and then you can use the table type as a parameter:

	CREATE OR REPLACE TYPE OracleInsightTestType AS OBJECT (x int, z int);
	CREATE OR REPLACE TYPE OracleInsightTestTable AS TABLE OF OracleInsightTestType;
	CREATE OR REPLACE PROCEDURE OracleInsightTestTableProc 
			(t OracleInsightTestTable, rs out sys_refcursor) 
		IS 
		BEGIN
			OPEN rs FOR SELECT * FROM TABLE(t); 
		END;

In .NET, you will need a few types. Fortunately, Insight provides a few base types to implement stuff for you. Note that you must create the factory names and bind the types to your database schema.type. Supposedly you can do this as part of a config file.

		public class TvpData : OracleCustomType
		{
			[OracleObjectMapping("X")]
			public int? X { get; set; }
			[OracleObjectMapping("Z")]
			public int Z { get; set; }

			public override void FromCustomObject(OracleConnection con, IntPtr pUdt)
			{
				if (X.HasValue)
					OracleUdt.SetValue(con, pUdt, 0, X);
				OracleUdt.SetValue(con, pUdt, 1, Z);
			}
		}

		[OracleCustomTypeMapping("SYSTEM.ORACLEINSIGHTTESTTYPE")]
		public class TvpDataFactory : OracleObjectFactory<TvpData>
		{
		}

		[OracleCustomTypeMapping("SYSTEM.ORACLEINSIGHTTESTTABLE")]
		public class TvpDataArrayFactory : OracleArrayFactory<TvpData>
		{
		}

After you have this mess, you can then use Insight to call the procedure:

	var list = new List<TvpData>();
	list.Add(new TvpData() { X = 1, Z = 2 });
	var results = _connection.Query("OracleInsightTestTableProc", new { t = list });

## Known Issues ##

* BulkCopy of XML columns doesn't work. This appears to be an issue with ODP.NET.
* ReliableConnection<OracleConnection> may not handle all of the transient error codes. There are about 18000 Oracle error codes. If you need one added, let me know.
* The Oracle driver maps Oracle ints to .NET decimal. Insight generally is pretty forgiving with type conversions, so you shouldn't have to worry, but when you use ExecuteScalar, it might throw an exception.

