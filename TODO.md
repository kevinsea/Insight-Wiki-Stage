# TODO #

A list of things that we want to add to Insight.Database:

- Async Open - currently Insight implements Async Query/Result, but blocks on opening connections. Upgrade Auto-Open/Async to asynchrnously open the connection.
- Assign identities on INSERT - when inserting records with an identity column, automatically return the identity values on the inserted records and assign them to your objects.