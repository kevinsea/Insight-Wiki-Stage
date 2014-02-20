You can also use the Insight mapping engine to send objects to SQL Server bulk copy. Just use the BulkCopy method:

	List<Beer> beer = new List<Beer>();
	beer.Add(new Beer() { Name = "Sly Fox IPA", Flavor = "yummy", OriginalGravity = 4.2m });
	beer.Add(new Beer() { Name = "Hoppopotamus", Flavor = "hoppy", OriginalGravity = 3.0m });

	Database.Connection().BulkCopy("Beer", beer);

The BulkCopy method will perform a BULK INSERT to write the data to the table. Insight will automatically map the objects to the table's schema.

## Configuring the Transfer ##

You can also configure the data transfer options, such as the number of records in a batch:

	Database.Connection().BulkCopy("Beer", beer, 
		configure: (InsightBulkCopy bulk) =>
		{
			bulk.BatchSize = 50;

			// InsightBulkCopy wraps the provider-specific BulkCopy object
			// you can get the provider object by:
			var sqlBulk = (SqlBulkCopy)bulk.InnerBulkCopy;
		});