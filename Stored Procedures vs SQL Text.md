# Stored Procedures vs. SQL Text #

Insight is happy to work with either Stored Procedures or SQL Text. The choice is up to you.

We recommend using Stored Procedures for a few reasons:

* You can tune the SQL on the server side without updating your object code.
* You can set security on the stored procedures at a granular level, rather than granting SELECT/INSERT/UPDATE/DELETE permissions to entire tables.
* You can see exactly how your database is accessed by looking at the stored procedures.
* Insight gets better schema information from SQL Server when you use stored procedures, so there is a little less type conversion than when you use SQL text.

That being said, most of the Insight extension method have two forms: XXX and XXXSql. The XXXSql version defaults the command type to CommandType.Text. The methods that support this are:

* ExecuteAsync / ExecuteSqlAsync
* QueryAsync / QuerySqlAsync
* Execute / ExecuteSql
* ExecuteScalar / ExecuteScalarSql
* GetReader / GetReaderSql
* Query / QuerySql

If a method doesn't have an XXXSql version, but does have a commandType parameter, simply pass in CommandType.Text for that parameter and it will work with your SQL Text.

## Non-SQL Server Providers ##
If you are using a data provider other than SQL Server, your provider might not have support for telling Insight the parameters to the stored procedures. If you want to use Insight with those providers, you have a few choices:

* Add some comments to the command text so that Insight knows the names of the parameters. (e.g. "FindBeer -- @Name @Flavor")
* Contribute some code to this project to make your database provider tell us the name and types of parameters.