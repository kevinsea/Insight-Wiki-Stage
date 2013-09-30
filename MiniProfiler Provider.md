# MiniProfiler Provider #

MiniProfiler is an awesome open-source project for profiling your running code. You can wrap your database connection in a ProfiledDbConnection and transparently use it with Insight.Database.

The only thing you need to do is install the MiniProfilerInsightDbProvider.

## Initializing the MiniProfilerInsightDbProvider ##

Before you can use Insight.Database with MiniProfiler, you must first install and register the provider in your application. The provider allows Insight to unwrap the database connection to get to the advanced features of the inner connection.

1. Install the Insight.Database.Providers.MiniProfiler package from NuGet.
2. Call MiniProfilerInsightDbProvider.RegisterProvider(). 

### Integration with MiniProfiler ###

One technique to include SQL profiling is to add in the following extension

	/// <summary>
	/// Syntax sugar for profiling using MiniProfiler.
	/// </summary>
	public static class SqlConnectionExtensions
	{
		/// <summary>
		/// Returns a profiled connection.
		/// </summary>
		/// <param name="sqlConnection"></param>
		/// <returns></returns>
		public static IDbConnection Profiled(this SqlConnection sqlConnection)
		{
			if (MiniProfiler.Current != null)
				return new ProfiledDbConnection(sqlConnection, MiniProfiler.Current);
			else
				return sqlConnection;
		}
	}

Then you can change your code to look like:

    // map a beer the stored procedure parameters
    Database
      .Connection()
      .Profiled()
      .Execute("InsertBeer", beer);

Alternatively, you can have the profiled connection injected for you automagically. 