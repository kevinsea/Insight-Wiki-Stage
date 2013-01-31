# TimeSpan and Time #

The .NET classes for date and time *almost* match up with the SQL data types, but there are a few quirks.

The good news is that they both define a "tick" as a 100 nanosecond interval.

The .NET classes:

* DateTime - represents a Date and a Time as "ticks since 1/1/0001".
* TimeSpan - represents a duration in ticks

The SQL classes:

* datetime - represents a Date and a Time as "days since 1/1/1900".
* smalldatetime - a datetime with smaller accuracy.
* datetimeoffset - a datetime with a timezone.
* datetime2 - a datetime with additional precision.
* time - represents the time of day as "ticks since midnight". Cannot be longer than a 24-hour period.
* date - represents a Date as "days since 1/1/1900".

## Date Conversions ##

Insight.Database automatically converts between .NET DateTime and the following classes:

* datetime/smalldatetime/datetimeoffset/datetime2 - change in precision
* date - ignores time component and only keeps the date component

## TimeSpan Conversions ##

Insight.Database automatically converts between .NET TimeSpan and the following classes:

* time - ignores the date component. An exception is thrown if a TimeSpan of greater than 24 hours is used.
* datetime/smalldatetime/datetimeoffset/datetime2 - TimeSpans are assumed to be "ticks since 1/1/1900".

Converting a .NET TimeSpan to a SQL datetime:

	01.02:03:04 => 1900-01-01.02:03:04

Converting a SQL datetime to a .NET TimeSpan:

	1970-8-7 02:00:00 => (70 years, 8 months, 7 days, 2 hours)

## Recommendations for storing long TimeSpans in SQL ##

Read: [Working with Time Spans and Durations in SQL Server](http://www.sqlteam.com/article/working-with-time-spans-and-durations-in-sql-server])

Since datetime is based on "days since 1/1/1900", SQL allows you to convert from ints to datetime.

	DECLARE @t [int] = 1
	DECLARE @datetime [datetime] = @t
	PRINT @datetime
	-- prints Jan  2 1900 12:00AM

	DECLARE @d1 [datetime] = '2/3/1982 04:05:06'
	DECLARE @d2 [datetime] = 1
	PRINT @d1 + @d2
	-- prints Feb  4 1982  4:05AM

Since you can add datetimes and SQL just adds the dates, you can use datetime as a long timespan.

That's why when Insight.Database converts from .NET TimeSpan to SQL datetime, it offsets by 1/1/1900. You can then roundtrip your Timespans like this test case:

	CREATE PROC TimeAdd @t [datetime], @add [datetime] AS SELECT @t + @add

	// make a time and a span and add them
	DateTime now = new DateTime (1970, 2, 1, 1, 0, 5);
	TimeSpan adjust = new TimeSpan(1, 0, 0);

	var time = connection.ExecuteScalar<DateTime>("TimeAdd", new { t = now, add = adjust });
	Assert.AreEqual(now + adjust, time);

The above also works if your proc uses the time type but the type parameter is limited to a 24-hour period.

	CREATE PROC TimeAdd @t [datetime], @add [time] AS SELECT @t + @add
