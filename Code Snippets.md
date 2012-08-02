# Code Snippets #

If you are using Visual Studio for development, you may want to use some of the handy code snippets that come with Insight.Database.

## Installing Snippets ##

1. First install the [Insight.Database NuGet Package](http://nuget.org/packages/Insight.Database).
1. Look in your packages\Insight.Database\tools folder for InsightCodeSnippets.vsi. Double click on it.
1. The Visual Studio Installer will prompt you to install the snippets in the right place.

## InsightRepo Snippet ##
This snippet creates an IRepository interface and a Repository implementation for the standard CRUD operations that are built into [Insight.Database.Schema](http://nuget.org/packages/Insight.Database.Schema) AutoProcs.

To use the snippet:

1. Open up a new C# class file.
1. Put your cursor within the namespace area and type `insightrepo`. Press tab twice.
1. Visual Studio will generate the template and allow you to modify class names and types within it.

Since the interface and class are partial, you can extend the IRepository and Repository classes in separate partial class files. This will allow you to regenerate the repository snippet as we update the implementations to handle additional types of standard queries.

## Other Snippets ##

* InsightForEach - generates a ForEach structure.
* InsightMultiResults - generates a structure to process multiple result sets.