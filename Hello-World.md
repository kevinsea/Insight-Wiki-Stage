Let's create a very simple example that puts all of the code together and works right out of the box.  We'll use a stored procedure that's in every MSSQL database since SQL 2008, [sp_stored_procedures](https://msdn.microsoft.com/en-us/library/ms190504.aspx).

1. Create a Console Application project called 'Insight_HelloWorld' 

1. Add Insight.Database (see [[Installing-Insight]])

1. Paste this code into program.cs:

    ``` C#
    using System.Collections.Generic;
    using System.Data.SqlClient;
    using Insight.Database;

    namespace Insight_HelloWorld
    {
        class Program
        {
            static void Main(string[] args)
            {
                var repo = new DatabaseInfoRepo();

                IList<SpStoredProcResult> procs = repo.GetStoredProcedures("dbo");

                // foreach (SpStoredProcResult proc in procs)
                //    repo.SaveStoredProcedure(proc);
            }
        }

        class DatabaseInfoRepo
        {
            string _connectionString = @"YourConnectionString";

            public IList<SpStoredProcResult> GetStoredProcedures(string schema)
            {
                var parms = new SpStoredProcParms() { sp_owner = schema };

                var conn = new SqlConnection(_connectionString);

                IList<SpStoredProcResult> result = conn.Query<SpStoredProcResult>("sp_stored_procedures", parms);

                return result;
            }

            //public void SaveStoredProcedure(SpStoredProcResult storeProcData)
            //{
            //    var conn = new SqlConnection(_connectionString);

            //    conn.Execute("[sp_StoredProcedureInsert]", storeProcData);
            //}

        }

        internal class SpStoredProcParms
        {
            internal string sp_name;
            internal string sp_owner;
            internal string sp_qualifier;
        }

        internal class SpStoredProcResult
        {
            internal string Procedure_Qualifier;
            internal string Procedure_Owner;
            internal string Procedure_Name;
        }

    }
   ```
1. Set your connection string in the DatabaseInfoRepo class
1. That's it!  Set a breakpoint at the end of main and run the code

Not sure how it all works?  The Connections, Execute, and Getting Results sections explain it all.

Want to add the ability to save this data to a table?  Continue to [[Persist World]].
