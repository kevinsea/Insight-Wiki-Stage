Insight makes it easy to execute SQL commands. No more SqlCommand class, no more Parameters.AddWithValue. Insight automatically maps parameters from an object to the command.

For example:

	Beer beer = new Beer() { Name = "IPA" };

	// map a beer the stored procedure parameters: @Name = beer.Name
	Database.Connection().Execute("InsertBeer", beer);

	// map an anonymous object to the stored procedure parameters: @Name = "IPA"
	Database.Connection().Execute("DeleteBeer", new { Name = "IPA" });

For stored procedures, Insight will automatically ask SQL Server for the parameters to the procedure, and create a mapping between the parameter object and the procedure. For details on this process, see [[Query Parameter Mapping]].

You can also execute SQL Text with the ExecuteSql method:

	Beer beer = new Beer() { Name = "IPA" };

	// map a beer the stored procedure parameters: @Name = beer.Name
	Database.Connection().ExecuteSql("INSERT INTO Beer (Name) VALUES (@Name)", beer);

	// map an anonymous object to the stored procedure parameters: @Name = "IPA"
	Database.Connection().ExecuteSql("DELETE FROM Beer WHERE Name = @Name", new { Name = "IPA" });

In this case, Insight will scan the command string for parameters (prefixed with @), and map the object's fields to the parameters.

Insight also supports ExecuteScalar and ExecuteScalarSql. These methods return the first column of the first row of the result set:

	int count = Database.Connection().ExecuteScalar<int>("CountBeer", new { Name = "IPA" });

	int count2 = Database.Connection().ExecuteScalarSql<int>(
		"SELECT COUNT(*) FROM Beer WHERE Name LIKE @Name",
		new { Name = "IPA" });

Note how these methods automatically convert the return type to a type you specify.

[[Auto-Open]] - BACK || NEXT- [[Executing SQL Commands]]
