
## Faster is Better ##

You shouldn't have to worry about your framework being fast. That's why we use generics wherever possible, and when we can't, we emit dynamic IL at runtime to bind parameters and result sets. The bindings are generated at the first call for each proc or recordset signature, and then reused for each subsequent call. These mini-methods have been hand-optimized to waste none of your precious cycles.

## Stored Procedures are Your Friends ##

We are firm believers that you should call your database through stored procedures. It allows you to have control over the API to your database, and gives you a place to abstract the storage and logic and change it at runtime. So methods like *Query* and *Execute* default to CommandType.StoredProcedure.

## Convenience is the New Reality ##

Let's be real. You're not going to use stored procedures for everything, and there are times where you don't even want to bother creating classes for *just this one task*. So we also provide *QuerySql* and *ExecuteSql* that favor text-based queries, and support dynamic objects as both inputs and outputs. We also support auto-open/close semantics for connections so you don't have to.

## Lightweight is Better ##

Don't make me work to do the easy things. That's why there are no mapping attributes or XML config files.  We assume you have access to the source code and probably the database schema. Insight.Database infers the mappings for your objects by matching up column names and property names. It can even infer one-to-one and one-to-many relationships for you. However, if Insight guesses wrong, there is always a way to give it a hint about the way you like to code.

## Object Code is the Priority ##

Insight.Database is designed to let you write your code as objects, passing them in and out with the least amount of boilerplate code. Code readability is a high priority. Boilerplate is bad. That's why we let you bind an interface definition directly to your database with zero lines of code. 

## Asynchronous is the Future ##

If you want to scale your system, your code needs to be asynchronous. But writing async database code is virtually impossible, particularly if you want to use an ORM. (And just try to figure out how to close your connection at *just* the right time.) Insight has full support for async code, as long as the database provider supports it.

## Batchy is Important Too ##

If you want to work in objects, but need high-performance, we can do that too. There are patterns to let you work on objects in result sets as they stream in rather than putting everything in memory at the same time. Did I mention bulk copy support too? Stream your objects into the database! Or get crazy and pull an object result set from one database, transform it, and bulk copy it into another database...all asynchronously!

## Tear at the Dotted Line ##

Insight.Database is a few magik black boxes put together, but we try to put the dotted lines in the right places. There is always a time when you just need to use one bit of the tool to solve a problem. You can use the result set reader separately from the Async extensions, or the command generator. In an emergency, you can also write your own record or query readers and still use the rest of the framework.