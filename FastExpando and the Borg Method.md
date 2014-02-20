Sometimes you need to send two objects as parameters to the database. Normally, you would need to combine the properties into a new anonymous object.

	Beer beer = new Beer() { Name = "Sly Fox IPA" };
	Glass glass = new Glass() { Ounces = 32 };

	Database.Connection().Execute("PourADraft",
		new
		{
			Name = beer.Name,
			Ounces = glass.Ounces
		});

For small numbers of parameters, this isn't too bad, but it gets unweildy for larger objects.

Fortunately, FastExpando makes it easy to turn an object into a FastExpando, and then expand the expando with more objects. We call this process "assimilation".

	FastExpando x = beer.Expand();
	x.Expand(glass);

Now `x` has Name, Flavor, OriginalGravity and Ounces properties.

	dynamic d = x;
	Console.WriteLine(d.Name);
	Console.WriteLine(d.Ounces);

There are lots of handy extension methods to let you chain this together:

	FastExpando x = beer.Expand(glass);

This makes our call to the database a whole lot cleaner:
	
	Database.Connection().Execute("PourADraft", beer.Expand(glass));

You can probably find lots of other uses for this.

NOTE: The Expand method only copies public fields and properties from the object. Also, it only (currently) copies members from the compiled type.

For example:

	Beer beer = new Lager() { Color = "golden" }; // lager is derived from beer
	dynamic x = FastExpando.FromObject(beer);

	Console.WriteLine(d.Name);		// valid
	Console.WriteLine(d.Color);		// not valid (currently)
	
