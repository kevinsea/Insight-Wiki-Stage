# Change Log #

## v1.1 ##
Coming soon to a NuGet repository near you!

v1.1 contains a few breaking changes. **bold** changes require code changes. *Italic* changes just require recompilation. (We might just call this v2.0 before pushing it to NuGet.)

* Added support for ColumnAttribute-based mapping.
* Added support for [[Dynamic Database Calls]] with a handy `connection.ProcName(params)` syntax.
* *FastExpando.FromObject and .Expand no longer take type arguments and now pull properties out of the object's runtime type, not compiled type*. 
* *Insight methods now return IList<T> instead of List<T> for future-proofing*. It's better for the environment, at the cost of losing .ForEach and .Sort. You'll cope. Or yell at Microsoft.
* **Changed AsyncXXX methods to XXXAsync for consistency**.


## v1.0.55 ##
* Fixed multi-class deserializer in jit-optimized scenarios.