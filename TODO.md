# TODO #

A list of things that we want to add to Insight.Database:

- Better Support for MS Transient Fault Handling Application Block ReliableSqlConnection. Would require some Async features to be added to the ReliableSqlConnection class.
- An adapter between IRetryStrategy and the MS Transient Fault Handling Application Block's RetryManager.

Code cleanup tasks:

- Update MiniProfiler (external project) to support Async in .NET 4.5 so we can remove some dependencies
