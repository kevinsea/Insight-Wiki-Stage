## Insight 6.0 ##

I've had some good ideas about the next build of Insight, and I wanted to get some opinions before I started down the path.

Feel free to comment inline with your initials, like:

* JW - I think I like vanilla

Or write a big note at the bottom.

## Design Goals ##

A reminder of our design goals:

* Fast and easy and simple.
* SQL should be written in SQL. You shouldn't try to twist your data access into a weird object model. If you're going to do anything crazy, write it in SQL, preferably a proc.
* If Insight can guess it right 90% of the time, then make it automatic.
* Any SQL generation needs to work across databases.
* Any methods should play nice with interface generation.

All of that being said...

A lot of standard SQL operations (i.e. CRUD) are pretty straightforward. You shouldn't have to write these.
(Of course, if you're running SQL Server, Insight.Database.Schema will even generate all of the procs for you (see AUTOPROCs))

## SQL Generation in Insight6 ##

I'm thinking that Insight6 could generate the SQL (i.e. not require a proc) for the standard operations:

	// insert one or more beers: INSERT INTO Beer (...) VALUES (...)
	conn.Insert(Beer beer);
	conn.InsertMany(IEnumerable<Beer> beer);

	// update one or more beers
	conn.Update(Beer beer);
	conn.UpdateMany(IEnumerable<Beer> beers);

	// delete one or more beers
	conn.Delete(Beer beer);
	conn.DeleteMany(IEnumerable<Beer> beers)

These are pretty simple to implement. Insight should be able to detect the columns and Primary Keys and make this happen.

* JB: I like this ^^^^

**What about XXXMany on other databases?** - currently InsertMany requires TVPs, which isn't supported on all databases.

It should also be able to handle a few more operations by key fields:

	// select the beer matching the key
	Beer beer = conn.Select<Beer>(object key);
	Beer beer = conn.Select<Beer>(new { Id = 4 });

	// delete a beer matching the key
	conn.Delete<Beer>(object key);
	conn.Delete<Beer>(new { Id = 4 });

* JB: Definitely to this ^^^^

This begs the question of whether we want to allow for arbitrary WHERE clauses in SelectMany or DeleteMany:

	// select all of the beer with a OG of 1
	IList<Beer> beer = conn.SelectMany<Beer>(new { Gravity = 1 });

	// delete all of the beers with an OG of 1
	conn.DeleteMany<Beer>(new { Gravity = 1 });

* JB: If it's not too hard, yes. I'm not sure how many people will know to use this.
* CH: Writing a WHERE clause as an object strikes me as a bit weird. 

An alternate version of this could use Expressions. This is nice, because then we can use **operators**.

	// operators for filtering!
	Beer beer = conn.Select<Beer>(b => b.Id == 4);
	IList<Beer> beer = conn.SelectMany<Beer>(b => b.Gravity > 1);
	conn.DeleteMany<Beer>(b => b.Gravity > 1);

Let me repeat here that I **don't** want this to turn into another way of writing SQL. I also don't want to rewrite Entity Framework. I just want to eliminate the repetitive SQL that you write.

* JB: **IF** this can be done so that it works in all cases (i.e. if I have a pretty complex expression), then okay. Again, I'm not sure how much it will be used. I'd rather you expand times on the Insight Schema management for other databases instead of this. If I had an alternative to migrations for PG and mySql **AND** I could use Insight end-to-end, that'd be a pretty huge win for me.
* CH: Expressions are less weird to write WHERE clauses. It would feel more natural to write it as `IList<Beer> beer = conn.SelectMany<Beer>().Where(b => b.Gravity > 1);` but that make is feel a lot like rewriting Entity Framework/Linq... Maybe expressions are a slippery slope. I can see the benefit of having simple WHERE clauses being represented in C#, I'm just not sure where the cut off point is where the WHERE clause is too complex and SQL should just be used.

## A Question ##

Pause here and think about whether this buys you anything more than using the AUTOPROC feature in Insight.Database.Schema. Should I focus my time on adding other database support to I.D.S?

## Joins ##

I suppose with only a little more work, we could support joins on SQL generated selects:

	// one-to-one syntax. Use the FK relationship to bring back the glass too.
	Beer beer = conn.Select<Beer, Glass>(b => b.ID = 4);

If I was smart, I would figure out a way to hook it into the QueryReader system, so you could do:

	// one-to-many syntax. Use the FK relationship to bring back two recordsets and let Insight merge them.
	Beer beer = conn.Select(b => b.ID = 4,
		Query.Returns(Some<Beer>.Records)
			.ThenChildren(Some<Chips>.Records));

Or:

	// not sure how I make this syntax work, but two queries in a single request...
	var results = conn.Select(b => b.ID = 4, customer => customer.ID = 5,
		Query.Returns(Some<Beer>.Records)
			.Then(Some<Customer>.Records));

* CH: It would be cool if it all hooked in and worked nicely with the QueryReader system

## Interfaces ##

Of course, this would all work with interfaces too:

	interface IBeerStore
	{
		Beer Select(Func<Beer, bool> where);

		Results<Beer, Customer> Select(Func<Beer, bool> where1, Func<Customer, bool> where2);
	}

Or a templated repository:

	interface IRepository<T>
	{
		T Select(Func<T, bool> where);
		...
	}

Either one could automatically generate the SQL.

* JB: Probably not a feature I'd use all that often. But that doesn't mean you shouldn't do it. I'd like to hear what others say.

## Performance ##

I'm pretty sure that the generated SQL would cache well.

## Unit of Work Pattern ##

I'm pretty sure that I don't want to go here, but let's have the discussion.

In a high-performance transactional system, you don't want to hold locks open for long. So this is bad:

	begin transaction
	get some records
	modify records
	update records
	...switch to another business object while records are locked...
	get some records (maybe some are the same)
	make more changes
	...switch to another business object while records are locked...
	...
	commit transaction

In this case, you probably want to send up your changes all at once, and a good way to do this is the Unit of Work pattern. In Entity Framework, you get this through the DbContext and change tracking.

	var context = DbContext();
	var objects = context.GetObjectsSomehow();
	objects.Modify();
	// switch to another business object, but use the same context
	context.GetMoreObjects().Modify();
	context.Save();

If we can reasonably generate the SQL because of the other things above, we would only need a few more pieces:

* Object Tracking
* Save makes a transactional update of all of the data.

At this point, we're not too far from having a fully-featured ORM, so it's tempting to attempt. At the same time, it can add a LOT of complexity and overhead.

* JB: Nonononono
* CH: I don't think I'd want a DbContext object. It would be neat if Insight did the right things automatically for me in terms of performance. 

## Squishy Feelings ##

These features feel neat, and I think I could do them without trashing the codebase. But...

* Expressions for where clauses make me worry that people will want to drag this into a full-featured SQL generator.
* Change-tracking has been known to make your head explode and/or your server overheat.

## Notes ##

Do you think you would use any/all of these features? Has anything else been nagging you?

Add your notes here.

* JB : One thing that would be helpful would be support for sorting. I'm not sure how it could work, but it's fairly common to want to arbitrarily change the sort order of a query.

* RP : I get where you are going with this, but I think Insight will cease to be a micro-ORM. What Insight does now is powerful and simplistic for the developer who uses it, but the risk here is that it becomes another SharePoint (does a lot of stuff, but none of it particularly well, or not as good as it once was). I also think that introducing a LinqProvider would be a wonderful challenge but it might just be a black-hole for your time. If you do go ahead with these features (which would be cool, don't get me wrong), I would prefer to see it packaged in isolation from the existing Insight packages.

* CH: I'd use the simple Select/Update. I'm not sure I'd really use more complex expressions, I'd rather write some of that SQL myself. 