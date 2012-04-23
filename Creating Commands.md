# Creating Commands #

Insight provides automatic mapping of objects to command parameters. You can use the CreateCommand extension methods to quickly convert your objects into DbCommands.

You probably won't use this directly unless you need to reuse the command for multiple calls to the database.

* For more information on how Insight maps query parameters, see [[Query Parameter Mapping]].
* For more information on additional common method parameters, see [[Common Method Parameters]].

## IDBConnection.CreateCommand ##
CreateCommand lets you create a DbCommand object from the name of a procedure and an object. Insight automatically maps the object parameters to the stored procedure parameters.

**NOTE: CreateCommand does NOT support auto-open, so you need to open the connection yourself.**


	using (SqlConnection connection = Database.Open())
	{
		IDbCommand command = connection.CreateCommand("FindBeer", new { Name = "IPA" });
	}

	// do something with the command

CreateCommandSql is the same, but defaults the command type to CommandType.Text.

	using (SqlConnection connection = Database.Open())
	{
		IDbCommand command = connection.CreateCommandSql("SELECT * FROM Beer WHERE Name = @Name", new { Name = "IPA" });
	}
