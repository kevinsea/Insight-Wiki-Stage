# MySql Provider #

The MySql Provider allows Insight.Database to work with your MySql database.

## Initializing the MySqlInsightDbProvider ##

Before you can use Insight.Database with MySql, you must first install and register the provider in your application. The provider allows Insight to handle some of the connection-specific database issues.

1. Install the Insight.Database.Providers.MySql package from NuGet.

## Known Issues ##

* Bulk Copy is not supported.
* Table Valued Parameters are not supported.
* XML columns are not supported.

## Examples of use ##

You'll most likely need to add the MySql.Data nuget package to your project as well. 

`Install-Package MySql.Data`

If you're using a web application, you'll probably want to add the RegisterProvider in the Application_Start of your Global.asa.cs file.

        using System;
        using System.Collections.Generic;
        using System.Linq;
        using System.Web;
        using System.Web.Mvc;
        using System.Web.Optimization;
        using System.Web.Routing;
        using Insight.Database.Providers.MySql;

        namespace MyWebProject
        {
            public class MvcApplication : System.Web.HttpApplication
            {
                protected void Application_Start()
                {
                    AreaRegistration.RegisterAllAreas();
                    FilterConfig.RegisterGlobalFilters(GlobalFilters.Filters);
                    RouteConfig.RegisterRoutes(RouteTable.Routes);
                    BundleConfig.RegisterBundles(BundleTable.Bundles);

                    // Insight Database changes.
                    MySqlInsightDbProvider.RegisterProvider();
                }
            }
        }

Then, instead of the SQL connection string builder, you can use this:

        var db = new MySqlConnectionStringBuilder("... your connection string ...");

And that will allow you to get your mySql beers:

       var beers = db
                   .Connection()
                   .As<IBarTender>()
                   .GetMeBeer(2)
                   .ToList();


