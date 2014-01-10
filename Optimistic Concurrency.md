# Optimistic Concurrency #

In some systems, it is important to check whether someone else has changed your data while you aren't looking.

For example:

1. Beer = SELECT Beer WHERE ID = 1 
2. Beer = Beer.Remaining * 0.5
3. Someone else drinks the whole beer while you aren't looking.
4. UPDATE Beer SET Remaining=0.5 WHERE ID = 1

This would result in infinite quantities of beer (awesome!), but not realistic, so you have to check. It usually goes something like this:

1. BEGIN TRANSACTION
	1. Beer = SELECT Beer WHERE ID = 1 
	2. Beer = Beer.Remaining * 0.5
	3. Someone else drinks the whole beer while you aren't looking.
	4. UPDATE Beer SET Remaining=0.5 WHERE ID = 1 AND Beer.Remaining = @oldvalue
	5. IF @@ROWCOUNT <> 0 ROLLBACK AND RAISERROR 'CONCURRENCY CHECK'

If you want to do concurrency checking with Insight.Database, it's not that hard.

First, make your SQL raise an error when you detect a concurrency exception. By default, Insight.Database will look for the string 'CONCURRENCY CHECK' in your exception.

	IF @@ROWCOUNT <> 0 BEGIN ROLLBACK; RAISERROR 'CONCURRENCY CHECK', 16, 1; END

Then, wrap your connection in an OptimisticConnection:

	var c = new OptimisticConnection(new SqlConnection(connectionString));

When the connection gets an exception, it checks for the text 'CONCURRENCY CHECK' and throws an OptimisticConcurrencyException. You can catch it:

	try
	{
		c.Execute("UpdateBeer", beer);
	}
	catch(OptimisticConcurrencyException ox)
	{
		// someone drank your beer
	}

## Insight.Database.Schema ##

If you are using Insight.Database.Schema and Autoprocs, you can enable Optimistic checks with the `Optimistic` flag:

	-- AUTOPROC All,Optimistic tablename

## Your Own Exceptions ##

If you want to raise the exceptions a different way, you can also derive your own class from OptimisticConnection and override IsConcurrencyException.