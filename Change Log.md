# Change Log #

## v1.1.4 ##

* Added caching to ObjectListDbReader to improve performance and eliminate an annoying exception that pops up while debugging.

## v1.1.2 ##

* Passing a single object to [[Dynamic Database Calls]] now results in the object's properties being mapped to parameters, just like with Execute and Query methods.

## v1.1.1 ##
Now in NuGet!

v1.1 contains a few breaking changes. **bold** changes require code changes. *Italic* changes just require recompilation. (We might just call this v2.0 before pushing it to NuGet.)

* Added support for ColumnAttribute-based mapping.
* Added support for [[Dynamic Database Calls]] with a handy `connection.ProcName(params)` syntax.
* *FastExpando.FromObject and .Expand no longer take type arguments and now pull properties out of the object's runtime type, not compiled type*. 
* *Insight methods now return IList<T> instead of List<T> for future-proofing*. It's better for the environment, at the cost of losing .ForEach and .Sort. You'll cope. Or yell at Microsoft.
* **Changed AsyncXXX methods to XXXAsync for consistency**.


## v1.0.55 ##
* Fixed multi-class deserializer in jit-optimized scenarios.