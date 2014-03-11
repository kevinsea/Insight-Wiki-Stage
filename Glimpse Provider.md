# Glimpse Provider #

Glimpse is an awesome open-source project for profiling your running code. You can wrap your database connection in a GlimpseDbConnection and transparently use it with Insight.Database.

The only thing you need to do is install the GlimpseInsightDbProvider.

## Initializing the GlimpseInsightDbProvider ##

Before you can use Insight.Database with Glimpse, you must first install and register the provider in your application. The provider allows Insight to unwrap the database connection to get to the advanced features of the inner connection.

## Using Insight and Glimpse ##

1. Install Insight.Database.Providers.Glimpse and Glimpse.Ado.
2. Wrap your DbConnection in a GlimpseDbConnection. Several options to do this:
	1. Glimpse automatically does it when you create a connection by providerName.
	2. Wrap it manually at runtime.
	3. Use your dependency injection to wrap it.

## A Magic Sample ##

Start with a standard MVC4 Application. Use a wizard.

Make sure your connection string is set in web.config. Note that we specify a providerName on the connection string. It tells the runtime code how to make the proper connection type.

	<connectionStrings>
		<add name="DefaultConnection" connectionString="Data Source = .; Trusted_Connection = true"
			 providerName="System.Data.SqlClient" />
	</connectionStrings>

Let's assume you want to put your database code in your controller. You could also put it in a business object or service.

In the controller constructor, we use the connection string settings and convert that to a DbConnection. Later in the action, we use the connection to execute some SQL or to convert it to an interface to call directly.

	public class HomeController : Controller
	{
		private IDbConnection _connection;

		public HomeController()
		{
			_connection = System.Configuration.ConfigurationManager
				.ConnectionStrings["DefaultConnection"]
				.Connection();
		}

		public ActionResult Index()
		{
			_connection.ExecuteSql("SELECT x = 1");
			_connection.ExecuteSql("SELECT y = @foo", new { foo = "bar" });

			var repo = _connection.As<IMyRepository>();
			repo.SelectBeer("ipa");

			return View();
		}
	}

Install Glimpse.Ado from Package Manager Console. This will also install the Glimpse core features.

	PM> Install-Package Glimpse.Ado

Notice that the web.config has a providerName that specifies the type of database connection to use. The magic of Glimpse will trap the request to create the provider "System.Data.SqlClient" and instead return you a SqlClient wrapped in a logging GlimpseDbProvider. All of the connections, etc. will be like scallops wrapped in bacon (yum).

Unfortunately at this point, Insight will yell at you and tell you that it doesn't have a provider for GlimpseDbConnection. That's ok, just install it from package manager:

	PM> Install-Package Insight.Database.Providers.Glimpse

Then in your startup code, say Global.asax.cs, register the provider:

	GlimpseInsightDbProvider.RegisterProvider();

Now Insight will know how to get at the real DbConnection inside of a GlimpseDbConnection and everyone will be happy. The nice part is that we didn't need to change any of our application code to make Insight and Glimpse work together.

## A Less Magic Sample ##

If you aren't using ConfigurationManager.ConnectionStrings to automatically create your database connection, you might need to make a little change.

Here, we are injecting an actual connection into the controller:

	public class HomeController : Controller
	{
		private IDbConnection _connection;

		public HomeController([Inject] connection)
		{
			_connection = connection;
		}

		public ActionResult Index()
		{
			_connection.ExecuteSql("SELECT x = 1");
			_connection.ExecuteSql("SELECT y = @foo", new { foo = "bar" });

			return View();
		}
	}

	// in your setup code
	Kernel.Bind<IDbConnection>().To(new SqlConnection(connectionString));

Now we have to wrap the SqlConnection in a GlimpseDbConnection before we inject it:

 	Kernel.Bind<IDbConnection>().To(new GlimpseDbConnection(new SqlConnection(connectionString)));

This isn't so bad as long as there is a central place you create your connections. For example, if you only inject repositories into your controllers:

	public class HomeController : Controller
	{
		private IMyRepository _repo;

		public HomeController([Inject] repo)
		{
			_repo = repo;
		}

		public ActionResult Index()
		{
			var beer = _repo.SelectBeer("ipa");

			return View();
		}
	}

	// your setup code has this:
	Kernel.Bind<IMyRepository>().To(new SqlConnection(connectionString).As<IMyRepository>());

All you would need to do is wrap your SqlConnection in a GlimpseDbConnection. It's probably wise to use a level of indirection:

	Kernel.Bind<DbConnection>().To(new SqlConnection(connectionString));
	Kernel.Bind<IMyRepository>().To(Kernel.Get<DbConnection>().As<IMyRepository>());

