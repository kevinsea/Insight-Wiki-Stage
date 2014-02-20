In v4.0, there was one major goal: 

	Making it easy to return arbitrary trees of objects from SQL results.

Before v4.0, you could return:

* Single Records
* Lists of Records
* Multiple Lists of Records

Each record could be:

* A single object.
* A single object with one-to-one mappings to additional records.

The big thing missing was:

* An object with a list of children (one-to-many).

And of course:

* An object with a list of children with a list of children.
* An object with a list of children with other one-to-one mappings.
* etc.

Now you can do all of this.

See [[Object Graphs]] to learn how. 