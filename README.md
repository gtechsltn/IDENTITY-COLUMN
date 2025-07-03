# IDENTITY COLUMN
+ ADO.NET
+ SQL Server
+ IDENTITY IDENTITY(1,1)
+ SCOPE_IDENTITY()

## SQL Server: Create Table with IDENTITY Column
```
CREATE TABLE Users (
    UserId INT IDENTITY(1,1) PRIMARY KEY,
    UserName NVARCHAR(100) NOT NULL
);
```

## SQL Server: Inserting Rows (No need to specify UserId)
```
INSERT INTO Users (UserName)
VALUES ('Alice'), ('Bob'), ('Charlie');
```

## SQL Server: Query the last identity value inserted in the current scope
```
SELECT SCOPE_IDENTITY();
```

## SQL Server: Insert Explicit ID (Override IDENTITY)
```
SET IDENTITY_INSERT Users ON;

INSERT INTO Users (UserId, UserName)
VALUES (1001, 'Admin');

SET IDENTITY_INSERT Users OFF;
```

## ADO.NET C# Code Snippet
```
using System;
using System.Data.SqlClient;

string connectionString = "your_connection_string";
int userId = 1001;
string userName = "john_doe";

using (var connection = new SqlConnection(connectionString))
{
    connection.Open();
    using (var transaction = connection.BeginTransaction())
    {
        try
        {
            var cmd = connection.CreateCommand();
            cmd.Transaction = transaction;

            // Turn ON IDENTITY_INSERT
            cmd.CommandText = "SET IDENTITY_INSERT Users ON";
            cmd.ExecuteNonQuery();

            // Insert with explicit ID
            cmd.CommandText = "INSERT INTO Users (UserId, UserName) VALUES (@UserId, @UserName)";
            cmd.Parameters.AddWithValue("@UserId", userId);
            cmd.Parameters.AddWithValue("@UserName", userName);
            cmd.ExecuteNonQuery();

            // Turn OFF IDENTITY_INSERT
            cmd.CommandText = "SET IDENTITY_INSERT Users OFF";
            cmd.Parameters.Clear();
            cmd.ExecuteNonQuery();

            transaction.Commit();
        }
        catch
        {
            transaction.Rollback();
            throw;
        }
    }
}
```

## ADO.NET Example: Insert & Read IDENTITY Value

### 1. Auto-insert (get generated IDENTITY value)

```
using System;
using System.Data.SqlClient;

string connStr = "your_connection_string_here";

using (var conn = new SqlConnection(connStr))
{
    conn.Open();

    var cmd = new SqlCommand(@"
        INSERT INTO Users (UserName)
        VALUES (@UserName);
        SELECT SCOPE_IDENTITY();", conn);

    cmd.Parameters.AddWithValue("@UserName", "David");

    var newId = Convert.ToInt32(cmd.ExecuteScalar());
    Console.WriteLine($"Inserted user with ID: {newId}");
}
```

### 2. Note about SCOPE_IDENTITY()
SCOPE_IDENTITY() returns the last identity value inserted in the current scope.
