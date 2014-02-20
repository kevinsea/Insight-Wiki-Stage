If you have existing code that you want to upgrade to v4.x:

1. Update Insight.Database to 4.x.
2. Everything should just compile, unless you used `DefaultGraph` or `Graph<T>`.
3. Convert `DefaultGraph` to `RecordsetAttribute`.
4. Convert `Graph<T>` to `Query.Returns`.

If you don't have time to update your code, but still want to use v4.x, you can try:

1. Install Insight.Database.Compatibility3x.

This will let you use some of the missing classes / extensions.

## Features No Longer Supported in v4.x ##

* DefaultGraph with more than one graph. This means you won't be able to tag an interface method that returns `Results` and has more than one one-to-one mapping. You will need to convert these to use `RecordsetAttribute`.
* Interface generation will no longer accept `withGraph` or `withGraphs` as parameters. I doubt anyone used this feature.