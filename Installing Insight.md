It's easy to get started with Insight.Database.

1. Download Insight.Database from [http://www.nuget.org/packages/Insight.Database](http://www.nuget.org/packages/Insight.Database).
2. If you're using a database other than SQL Server, get the proper provider package (e.g. Insight.Database.Providers.MySql.
3. Import the extensions into your code by adding `using Insight.Database;`
4. Register the provider for your database (see below). 
5. Write code.

## Registering a Provider ##

Insight will attempt to auto-register any provider DLLs it finds in your application search path or in the same folder as the Insight.Database assembly.

However, for some project configurations (e.g. add-ins or class libraries created in another folder), auto-registration may fail to find the provider assembly. You'll get an exception that says:

	Have you loaded the provider for your database?

If you get this in your project, you can force Insight to load a particular provider by calling the RegisterProvider method on the database provider.

For example:

	using Insight.Database;

	// at some point in application startup...
	SqlInsightDbProvider.RegisterProvider();

Or:

	using Insight.Database.Providers.MySql;

	// at some point in application startup...
	MySqlInsightDbProvider.RegisterProvider();

See [[Insight and Data Providers]] for the list of providers.
