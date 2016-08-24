Let's extend the [[Hello World]] example to do a save:

1. First let's create a table and a procedure to save the data

    ```SQL
	Create Table dbo.[StoredProcedure]
	(
		Procedure_Name		[nvarchar](500) NULL,
		Procedure_Owner		[nvarchar](500) NULL,
		Procedure_Qualifier	[nvarchar](500) NULL
	)
	GO

	Create Procedure dbo.[sp_StoredProcedureInsert] 
		  @procedure_name		[nvarchar](500)
		, @procedure_owner		[nvarchar](500)
		, @procedure_qualifier	[nvarchar](500)

	AS 
	BEGIN
		INSERT INTO [StoredProcedure]
				   ([Procedure_Name], [Procedure_Owner], [Procedure_Qualifier])
			 VALUES (@procedure_name, @procedure_owner, @procedure_qualifier)
	END

	GO
    ```

1.  Now uncomment the following code blocks in the [[Hello World]] example

    ```C#

        foreach (SpStoredProcResult proc in procs)
            repo.SaveStoredProcedure(proc);

    ```
    and

    ```C#
        public void SaveStoredProcedure(SpStoredProcResult storeProcData)
        {
            var conn = new SqlConnection(_connectionString);
    
            conn.Execute("[sp_StoredProcedureInsert]", storeProcData);
        }
    ```

1. That's it!  Set a breakpoint at the end of main and run the code

Note that as you read on you will see it quite easy to pass a whole list of objects to a stored procedure.
