# Mapping Results to Objects #

Insight analyzes each result set and creates a mapping between the result set and the desired return types.

1. Analyze the schema of the result set to determine the fields that have been returned.
1. Determine the available fields and properties that can be set.
1. Build a mapping between database fields and object properties.
1. Cache the deserializer for the result set so it can be reused for any mapping that matches the signature. The  signature of the result set is a hash of the column names and the data types.

## Analyze the Schema of the Result Set ##
Insight calculates a SchemaIdentity for each result set. This is a hash of the columns and data types and is expected to be distinct for any practical sets of recordsets.

## Determine the Available Properties ##
Insight uses reflection to determine the fields and properties on the desired return types. Any public, internal, or private field or property can be used as a binding target.

This information is cached for subsequent mapping operations.

## Build a Mapping Between Database and Object ##
Insight maps the database fields to object members using a case-insensitive exact match.

The mapper can also handle special types (see [[Xml Parameters and Results]]) as well as object hierarchies (see [[Object Hierarchies]]).