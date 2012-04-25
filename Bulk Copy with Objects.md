# Bulk Copy with Objects #

You can also use the Insight mapping engine to send objects to SQL Server bulk copy. Just use the BulkCopy method:

	List<Beer> beer = new List<Beer>();
	beer.Add(new Beer() { Name = "Sly Fox IPA", Flavor = "yummy", OriginalGravity = 4.2m });
	beer.Add(new Beer() { Name = "Hoppopotamus", Flavor = "hoppy", OriginalGravity = 3.0m });

	Database.Connection().BulkCopy("Beer", beer);

## batchSize Parameter ##

You can also control the batch size with the batchSize named parameter.

	Database.Connection().BulkCopy("Beer", beer, batchSize: 50);