For the SQL Server provider, you can use the `xml` data type as both input parameters and results. Insight automatically can convert various types to and from the `xml` type.

Assume you have a stored procedure like this:

	CREATE PROC InsightTestProc (@Xml xml) AS 
		-- do something
	GO

## XmlDocument as an Xml Parameter ##
XmlDocument is converted to an Xml parameter by sending the OuterXml to the server.

	// create a document
	XmlDocument doc = new XmlDocument();
	doc.LoadXml("<Data><Text>foo</Text></Data>");

	_connection.Execute("InsightTestProc", new { Xml = doc });

## XDocument as an Xml Parameter ##
XDocument is also converted to an Xml parameter by sending the OuterXml to the server.

	XDocument doc = XDocument.Parse("<Data><Text>foo</Text></Data>");

	_connection.Execute("InsightTestProc", new { Xml = doc });

## String as an Xml Parameter ##
Strings are sent unconverted to an Xml parameter. The caller is required to verify that the string can be converted to Xml by the server.

	string doc = "<Data><Text>foo</Text></Data>";

	_connection.Execute("InsightTestProc", new { Xml = doc });

## Object as an Xml Parameter ##
Any other object is converted to an Xml parameter by using the DataContractSerializer.

	class Data
	{
		public string Text;
	}

	// create a document
	Data d = new Data()
	{
		Text = "foo"
	};

	_connection.Execute("InsightTestProc", new { Xml = d });

## Xml Columns in Table-Valued Parameters ##
Yes, this works too.

Assume you have:

	CREATE TYPE [XmlDataTable] AS TABLE ([Data] [Xml])
	CREATE PROCEDURE [XmlTestProc] @p [XmlDataTable] READONLY AS SELECT * FROM @p

When Insight maps an object to XmlDataTable, it will automatically use the rules above to convert the property into Xml for the column.

So if you have an object that contains an object:

	class Parent
	{
		public Data Data;
	}

You can send a Parent to the database, and Data will automatically be converted to the Xml column using the DataContractSerializer.

	Parent r = new Parent();
	r.Data = new Data();
	r.Data.Text = "foo";

	var list = _connection.Query<Parent>("XmlTestProc", new { p = new List<Parent>() { r } });
	var item = list[0];

Note that the query results also convert Xml columns back into objects, XDocuments, XmlDocuments or strings for you.

Really? Yes.