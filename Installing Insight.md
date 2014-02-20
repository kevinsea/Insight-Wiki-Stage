It's very easy to get started with Insight.Database.

1. Download Insight.Database from [http://www.nuget.org/packages/Insight.Database](http://www.nuget.org/packages/Insight.Database).
1. Import the extensions into your file by adding `using Insight.Database;`.
1. Write code.

If you are using one of the Database Providers not included in the base install you also need to install that package and register the provider in your application startup code. For example:

	// in application startup...	
	MySqlInsightDbProvider.RegisterProvider();

See [[Insight and Data Providers]] for the list of providers.
