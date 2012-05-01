# Overriding Mapping with ColumnAttribute #

In some rare cases, you may need to override the default column mapping logic in Insight. By default, a database column is mapped to a field or property with the same name. If you don't have access to the database to create a stored procedure or view, you may be forced to deal with it in your code.

Rule #1: try to fix it with a stored procedure or a view. If the database has bad names for things, it's probably going to confuse something else, so try to fix it at the source.

Rule #2: see Rule #1.

If that doesn't work, you can add a ColumnAttribute to your data class to override the mapping.

	class Beer
	{
		[Column("Name")]
		public string BrandName;
	}

In this case, the `Name` column in the result set is sent to the BrandName field on your object.

In the case of conflicts, Insight follows these rules:

1. Fields are processed first, in order of appearance. (Well, in the order that reflection gives us the fields.)
1. Properties are processed next, in order of appearance.

If two members are mapped to the same database column, fields have precedence over properties and then the first defined field or property wins.

Rule #3: don't create ambigious mappings. You are just asking for trouble. 