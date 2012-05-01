# Change Log #

## v1.1 ##
Coming soon to a NuGet repository near you!

* FastExpando.FromObject and .Expand no longer take type arguments and now pull properties out of the object's runtime type, not compiled type. **You will need to re-compile your code with this change, but you should not need to modify any code. The compiler should take care of it all.** Sorry for the semi-breaking change...
* Insight methods now return IList<T> instead of List<T> for future-proofing. It's better for the environment, at the cost of losing .ForEach and .Sort. You'll cope. Or yell at Microsoft.
* Added support for ColumnAttribute-based mapping.

## v1.0.55 ##
* Fixed multi-class deserializer in jit-optimized scenarios.