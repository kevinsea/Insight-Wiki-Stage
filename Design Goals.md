# Design Goals #
This section attempts to explain the design philosophy behind the Insight.Database library.

## Faster is Better ##
Insight.Database emits dynamic IL at runtime to bind the objects to parameters or result sets. the bindings are generated at the first call for each signature, and then reused. This keeps the runtime performance as fast as possible. Kudos to the Dapper team ([http://code.google.com/p/dapper-dot-net/](http://code.google.com/p/dapper-dot-net/)) for showing how this could be done efficiently.

## Stored Procedures are Your Friends ##
We are firm believers that you should call your database through stored procedures. It allows you to have control over the API to your database, and gives you a place to abstract the storage and logic and change it at runtime. So methods like *Query* and *Execute* default to CommandType.StoredProcedure.

## Convenience is the New Reality ##
Let's be real. You're not going to use stored procedures for everything, and there are times where you don't even want to bother creating classes for *just this one task*. So we also provide *QuerySql* and *ExecuteSql* that favor text-based queries, and support dynamic objects as both inputs and outputs. We also support auto-open/close semantics for connections so you don't have to. Just wait until we figure out how to integrate it with PowerShell...

## Lightweight is Better ##
Don't make me work to do the easy things. That's why there are no mapping attributes, or XML config files.  We assume you have access to the source code and probably the database schema. Insight.Database infers the mappings for your objects by matching up column names and property names. OK, so it also supports multi-object result sets, object hierarchies, and other complex patterns, but we try to keep it simple. (We'll probably support a little bit of mapping when we figure out how to do it without going down the slippery slope of config files.)

## Object Code is the Priority ##
Insight.Database is designed to let you write your code as objects, passing them in and out with the least amount of boilerplate code. Code readability is a high priority.

## Asynchronous is the Future ##
If you want to scale your system, your code needs to be asynchronous. But writing async database code is virtually impossible, particularly if you want to use an ORM. (And just try to figure out how to close your connection at *just* the right time.) Insight has full support for async code, as long as the database provider supports it (so far this is just SQL Server). And when C# 4.5 goes mainstream, you are going to want this.

## Batchy is Important Too ##
If you want to work in objects, but need high-performance, we can do that too. There are patterns to let you work on objects in result sets as they stream in rather than putting everything in memory at the same time. Did I mention bulk copy support too? Stream your objects into the database! Or get crazy and pull an object result set from one database, transform it, and bulk copy it into another database...all asynchronously!

## Tear at the Dotted Line ##
Insight.Database is a few magik black boxes put together, but we try to put the dotted lines in the right places. There is always a time when you just need to use one bit of the tool to solve a problem. You can use the result set reader separately from the Async extensions, or the command generator.