Insight does a lot of magic to map objects to the parameters of a query. It takes the following steps:

1. Determine the parameters of the query.
1. Determine the properties of the parameter object. This includes **internal and private** fields and properties.
1. Create a mapping between them by name.
1. Generate IL to build a list of parameters from an instance of the object. This function is cached for future calls.

## Determining Query Parameters ##

If the Query is a StoredProcedure call Insight will use the provider to derive the parameters and determine the name and type of each parameter. This is the most efficient method, as it allows Insight to properly convert parameters to the proper type before sending it to the database. It also allows Insight to use the proper schemas for table-valued parameters.

Otherwise, Insight looks at the text of the Query for parameters that match the parameter regex `[?@:]([a-zA-Z0-9_]+)`. Therefore you can use `@Param` or `?Param` syntax, depending on your database provider. These parameters are assumed to be text values and are converted at the database. Note that this can cause some conversion issues with certain non-native types.

## Determining Properties of the Parameter Object ##

If the parameter object is a DynamicObject (e.g. `dynamic` or `FastExpando`), then the available properties are any property of the DynamicObject. Note that the DynamicObject must support the `IDictionary<string, object>` interface.

Otherwise, Insight uses reflection to find all of the public, internal, and private members of the parameter object. It reflects against the actual type of the object, so even if you use a variable of a "BaseClass", but pass in a value of "DerivedClass", the properties will reflect off the "DerivedClass".

## Create a Mapping from Property to Parameter ##

The mapping is performed from an exact case-insensitive string match for the property to the parameter.

You can also override the default mapping logic. See [[Customizing Object Mapping]].

## Special Case: Single Enumerable ##

If you pass an IEnumerable<T> in as the parameter object, Insight will assume that you want to turn it into a table-valued parameter. So it will look at the parameter list to find a TVP, and map the enumerable to that parameter. See [[Identity Inserts]] for an example.

## IL Generation ##

Insight uses dynamic IL method generation to efficiently access private properties and fields, and to provide the fastest performance.

[[Output Parameters]] - BACK || NEXT- [[Mapping Results to Objects]]
