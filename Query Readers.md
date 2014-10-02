When Insight goes to translate a bunch of recordsets into query results, it uses an instance of `IQueryReader<T>`. As you might expect, QueryReaders know how to read the results of queries.

There are several implementations:

* `SingleReader<T>` - reads a single record from the recordset.
* `ListReader<T>` - reads a list of records from the recordset.
* `ResultsReader<T1, T2...>` - reads multiple records from the recordset.

Each of the query readers can take an IRecordReader as a parameter. The record reader reads individual records within the query reader. That's how we combine one-to-one relationships with other types of reads.

The built-in readers can also be chained together to form complex structures. That's what the `Then` and `ThenChildren` methods do.

	ListReader<Beer> lr = Query.Returns(Some<Beer>.Records);
	RecordReader<Beer, Wine> rr = lr.Then(Some<Wine>.Records);
	RecordReader<Beer, Wine> rr2 = rr.ThenChildren(Some<Glass>.Records);
	
You can combine them into almost any structure you could need.

## Custom Query Readers ##

If there is some other structure you would need, you could always implement IQueryReader yourself. The rest of Insight will still work for you.

[[Record Readers]] - BACK || NEXT- [[Auto Interface Implementation]]