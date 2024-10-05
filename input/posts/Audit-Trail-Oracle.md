Title: Using an audit trail table on Oracle
Published: 05/10/2024 08:00
Tags: [CSharp, Oracle, Database] 
---

# Implementing Auditable Updates in a Relational Database

In modern applications, maintaining an audit trail of changes to data is crucial for compliance, debugging, and data integrity. This blog post explores a straightforward approach to implementing auditable updates in a relational database system, specifically focusing on a project management scenario with hierarchical data.

## Problem Description

We have a relational database containing ```Projects```, each of which includes ```Instruments```, ```Markets```, and ```Valuations```. These entities form a tree structure, adhering to the third normal form (3NF). Previously, any update to a project involved downloading the entire project tree, making changes, and uploading a new project under a new ID to ensure complete auditability.

This approach is inefficient for small updates and doesn't allow for granular tracking of changes. The goal is to enable small, precise updates to projects while maintaining a comprehensive audit trail of all changes.

## Solution Overview

We introduce an audit table that records every change made to the database. The audit table will store serialized JSON representations of operations like ```update```, ```insert```, and ```delete```. We'll also provide C# code to apply and revert these changes, effectively creating an undo stack.

Let's use the following DB Schema for illustration: 


![TableSchema](/posts/images/audit-trail/TableStructureBlog.png){ width = 60% }

- Primary Keys: Each table has a primary key (e.g., ```ProjectID```, ```InstrumentID```).
- Foreign Keys: Child tables reference their parent via foreign keys (e.g., ```instruments.ProjectID``` references ```projects.ProjectID```).
- Audit Table: The ```change_audit``` table records changes with fields like ```ChangeAuditID```, ```TimeApplied```, and ```ImpactJson```.

<!-- 
```d2
projects: {
  shape: sql_table
  ProjectID: int {constraint: primary_key}
  Name: varchar(100)
  Description: text
  LastUpdated: timestamp with time zone
  VersionNumber: int
}

instruments: {
  shape: sql_table
  InstrumentID: int {constraint: primary_key}
  ProjectID: int {constraint: foreign_key}
  Name: varchar(100)
  Type: varchar(50)
  LastUpdated: timestamp with time zone
}

markets: {
  shape: sql_table
  MarketID: int {constraint: primary_key}
  ProjectID: int {constraint: foreign_key}
  Region: varchar(50)
  MarketType: varchar(50)
  LastUpdated: timestamp with time zone
}

valuations: {
  shape: sql_table
  ValuationID: int {constraint: primary_key}
  ProjectID: int {constraint: foreign_key}
  Value: decimal(10, 2)
  Currency: varchar(10)
  LastUpdated: timestamp with time zone
}

change_audit: {
  shape: sql_table
  ChangeAuditID: int {constraint: primary_key}
  TimeApplied: timestamp with time zone
  UserID: varchar(100)
  ImpactJson: jsonb
}

instruments.ProjectID -> projects.ProjectID
markets.ProjectID -> projects.ProjectID
valuations.ProjectID -> projects.ProjectID

```
-->

## Implementing Change Auditing

The ```change_audit``` table is designed to store all changes in a JSON format for flexibility and ease of storage.

```sql 
CREATE TABLE change_audit (
  ChangeAuditID   NUMBER PRIMARY KEY,
  TimeApplied     TIMESTAMP,
  UserID          VARCHAR2(100),
  ImpactJson      CLOB
);
```

## JSON Structure for Changes
Each change is recorded as a JSON object:

```json 
{
  "Operation": "update",
  "impact": [
    {
      "Table": "Instruments",
      "PrimaryKey": {"ProjectID": 4, "InstrumentID": 2},
      "Column": "Name",
      "OldValue": "Old Instrument Name",
      "NewValue": "Updated Instrument Name"
    }
  ]
}
```

## CSharp to apply changes given an operation 

To apply changes recorded in the JSON, we'll use C# code that parses the JSON and executes the corresponding SQL commands.

I assume you have the ```_connectionString``` available somewhere as a constant in the code. 

```csharp
using Oracle.ManagedDataAccess.Client;
using Newtonsoft.Json.Linq;
using System;
using System.Collections.Generic;

public class ChangeApplier
{

    public void ApplyChanges(string jsonInput)
    {
        // Parse the JSON input
        var operation = JObject.Parse(jsonInput);
        string opType = operation["Operation"].ToString();
        var impactList = (JArray)operation["impact"];

        using (var conn = new OracleConnection(_connectionString))
        {
            conn.Open();
            using (var transaction = conn.BeginTransaction())
            {
                try
                {
                    foreach (var impact in impactList)
                    {
                        string table = impact["Table"].ToString();
                        var primaryKey = (JObject)impact["PrimaryKey"];
                        string column = impact["Column"]?.ToString();
                        string newValue = impact["NewValue"]?.ToString();

                        switch (opType)
                        {
                            case "update":
                                ApplyUpdate(conn, table, primaryKey, column, newValue);
                                break;
                            case "insert":
                                ApplyInsert(conn, table, impact);
                                break;
                            case "delete":
                                ApplyDelete(conn, table, primaryKey);
                                break;
                        }
                    }

                    transaction.Commit();
                }
                catch (Exception ex)
                {
                    transaction.Rollback();
                    Console.WriteLine($"Error applying changes: {ex.Message}");
                }
            }
        }
    }

    private void ApplyUpdate(OracleConnection conn, string table, JObject primaryKey, string column, string newValue)
    {
        var pkConditions = BuildPrimaryKeyCondition(primaryKey);
        var query = $"UPDATE {table} SET {column} = :newValue WHERE {pkConditions}";

        using (var cmd = new OracleCommand(query, conn))
        {
            cmd.Parameters.Add(new OracleParameter("newValue", newValue));
            cmd.ExecuteNonQuery();
        }
    }

    private void ApplyInsert(OracleConnection conn, string table, JToken impact)
    {
        var primaryKey = (JObject)impact["PrimaryKey"];
        var newValues = (JObject)impact["NewValues"];
        var columns = new List<string>();
        var values = new List<string>();

        foreach (var property in primaryKey.Properties())
        {
            columns.Add(property.Name);
            values.Add($":{property.Name}");
        }

        foreach (var property in newValues.Properties())
        {
            columns.Add(property.Name);
            values.Add($":{property.Name}");
        }

        var query = $"INSERT INTO {table} ({string.Join(", ", columns)}) VALUES ({string.Join(", ", values)})";

        using (var cmd = new OracleCommand(query, conn))
        {
            foreach (var property in primaryKey.Properties())
            {
                cmd.Parameters.Add(new OracleParameter(property.Name, property.Value.ToString()));
            }

            foreach (var property in newValues.Properties())
            {
                cmd.Parameters.Add(new OracleParameter(property.Name, property.Value.ToString()));
            }

            cmd.ExecuteNonQuery();
        }
    }

    private void ApplyDelete(OracleConnection conn, string table, JObject primaryKey)
    {
        var pkConditions = BuildPrimaryKeyCondition(primaryKey);
        var query = $"DELETE FROM {table} WHERE {pkConditions}";

        using (var cmd = new OracleCommand(query, conn))
        {
            cmd.ExecuteNonQuery();
        }
    }

    private string BuildPrimaryKeyCondition(JObject primaryKey)
    {
        var conditions = new List<string>();
        foreach (var prop in primaryKey.Properties())
        {
            conditions.Add($"{prop.Name} = :{prop.Name}");
        }
        return string.Join(" AND ", conditions);
    }
}
```
- **ApplyChanges**: Parses the JSON input and determines the operation type.
- **ApplyUpdate**: Executes an UPDATE SQL command using parameters to prevent SQL injection.
- **ApplyInsert**: Executes an INSERT SQL command, constructing columns and values from the JSON.
- **ApplyDelete**: Executes a DELETE SQL command based on the primary key.
BuildPrimaryKeyCondition: Constructs the WHERE clause for SQL commands.

A side note, for the insert, you'll have the challenge if you are using auto-incremented IDs, this will mean you don't know the new IDs until you have inserted the data, so you should make sure to capture the new IDs and then create the audit log. This is left as a simple exercise to the reader in case it is necessary.

## CSharp to revert changes 

To revert changes (undo operations), we'll process the audit trail in reverse order. Here I give the processing of a list of operations as an example of unrolling. It is to note that the reverse delete does only one table, so if there was some connected information that was deleted via referential identity, it was the task of the audit table to keep that in the audit. 

```csharp
public class ChangeReverter
{
    public void RevertChanges(List<string> jsonOperations)
    {
        using (var conn = new OracleConnection(_connectionString))
        {
            conn.Open();
            using (var transaction = conn.BeginTransaction())
            {
                try
                {
                    jsonOperations.Reverse(); // note: you could also have provided sorted by last time from the audit table instead of reversing them

                    foreach (var operationJson in jsonOperations)
                    {
                        var operation = JObject.Parse(operationJson);
                        string opType = operation["Operation"].ToString();
                        var impactList = (JArray)operation["impact"];

                        foreach (var impact in impactList)
                        {
                            string table = impact["Table"].ToString();
                            var primaryKey = (JObject)impact["PrimaryKey"];
                            string column = impact["Column"]?.ToString();
                            string oldValue = impact["OldValue"]?.ToString();

                            switch (opType)
                            {
                                case "update":
                                    RevertUpdate(conn, table, primaryKey, column, oldValue);
                                    break;
                                case "insert":
                                    ApplyDelete(conn, table, primaryKey);
                                    break;
                                case "delete":
                                    RevertDelete(conn, table, impact);
                                    break;
                            }
                        }
                    }

                    transaction.Commit();
                }
                catch (Exception ex)
                {
                    transaction.Rollback();
                    Console.WriteLine($"Error reverting changes: {ex.Message}");
                }
            }
        }
    }

    private void RevertUpdate(OracleConnection conn, string table, JObject primaryKey, string column, string oldValue)
    {
        var pkConditions = BuildPrimaryKeyCondition(primaryKey);
        var query = $"UPDATE {table} SET {column} = :oldValue WHERE {pkConditions}";

        using (var cmd = new OracleCommand(query, conn))
        {
            cmd.Parameters.Add(new OracleParameter("oldValue", oldValue));
            cmd.ExecuteNonQuery();
        }
    }

    private void RevertDelete(OracleConnection conn, string table, JToken impact)
    {
        var primaryKey = (JObject)impact["PrimaryKey"];
        var oldValues = (JObject)impact["OldValues"];
        var columns = new List<string>();
        var values = new List<string>();

        foreach (var property in primaryKey.Properties())
        {
            columns.Add(property.Name);
            values.Add($":{property.Name}");
        }

        foreach (var property in oldValues.Properties())
        {
            columns.Add(property.Name);
            values.Add($":{property.Name}");
        }

        var query = $"INSERT INTO {table} ({string.Join(", ", columns)}) VALUES ({string.Join(", ", values)})";

        using (var cmd = new OracleCommand(query, conn))
        {
            foreach (var property in primaryKey.Properties())
            {
                cmd.Parameters.Add(new OracleParameter(property.Name, property.Value.ToString()));
            }

            foreach (var property in oldValues.Properties())
            {
                cmd.Parameters.Add(new OracleParameter(property.Name, property.Value.ToString()));
            }

            cmd.ExecuteNonQuery();
        }
    }

    // Reuse BuildPrimaryKeyCondition and ApplyDelete methods from ChangeApplier
}
```

- **RevertChanges**: Processes the list of JSON operations in reverse order to undo changes.
- **RevertUpdate**: Sets the column back to its old value.
- **RevertDelete**: Re-inserts a deleted row using the old values stored in the audit trail.
- **ApplyDelete**: Deletes a row, used here to undo an insert operation.

## JSON schema

The reason that I prefer to use the Json directly in the C# code is that actually making up the C# classes for this schema is actually more work that processing the json directly in the code. 

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "ImpactJsonRoot",
  "type": "object",
  "properties": {
    "Operation": {
      "type": "string",
      "enum": ["update", "insert", "delete"],
      "description": "Type of operation"
    },
    "Impact": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "Table": {
            "type": "string",
            "description": "Name of the table affected"
          },
          "PrimaryKey": {
            "type": "object",
            "description": "Primary key fields and their values",
            "additionalProperties": {
              "type": ["number", "null"]
            }
          },
          "Column": {
            "type": "string",
            "description": "Column affected (for updates)"
          },
          "OldValue": {
            "type": ["string", "number", "boolean", "null"],
            "description": "Previous value (for updates and deletes)"
          },
          "NewValue": {
            "type": ["string", "number", "boolean", "null"],
            "description": "New value (for updates and inserts)"
          },
          "OldValues": {
            "type": "object",
            "description": "All old values (for deletes)",
            "additionalProperties": {
              "type": ["string", "number", "boolean", "null"]
            }
          },
          "NewValues": {
            "type": "object",
            "description": "All new values (for inserts)",
            "additionalProperties": {
              "type": ["string", "number", "boolean", "null"]
            }
          }
        },
        "required": ["Table", "PrimaryKey"]
      }
    }
  },
  "required": ["Operation", "Impact"]
}
```

and here are examples of operations: 

### update 

```json 
{
  "Operation": "update",
  "Impact": [
    {
      "Table": "Instruments",
      "PrimaryKey": { "ProjectID": 4, "InstrumentID": 2 },
      "Column": "Name",
      "OldValue": "Old Instrument Name",
      "NewValue": "Updated Instrument Name"
    }
  ]
}

```

### insert 

```json 
{
  "Operation": "insert",
  "Impact": [
    {
      "Table": "Instruments",
      "PrimaryKey": { "ProjectID": 4, "InstrumentID": 10 },
      "NewValues": {
        "Name": "New Instrument",
        "Type": "Flexible Asset",
        "LastUpdated": "2024-10-05T12:34:56Z"
      }
    }
  ]
}
```

### delete 

```json 
{
  "Operation": "delete",
  "Impact": [
    {
      "Table": "Instruments",
      "PrimaryKey": { "ProjectID": 4, "InstrumentID": 5 },
      "OldValues": {
        "Name": "Obsolete Instrument",
        "Type": "Flexible Asset",
        "LastUpdated": "2024-10-01T09:15:00Z"
      }
    }
  ]
}
```


Note: OpenAI's ```o1-preview``` was used to assist in the creation of the post. 