Title: Using bulk inserts on databases
Published: 11/11/2019
Tags: [CSharp, Oracle, MsSql, Postgresql, BulkInserts, Database] 
---
When inserting large amounts of data into databases, you should consider using the bulk functions instead of running single row INSERT commands. Single row INSERT command will usually lead to poor performance. There exists two main techniques for mutiple row insertions in databases from code and several more if you use the loader tools that accompany the database. Here I'll detail the two main methods: BulkInserts and VectorInserts. 

# Bulk Inserts for Oracle, MSSQL, Postgresql
For the examples, it is assumed that there exists an object Timeseries ts with a field IEnumerable TimeseriesData that contains Datapoints. A Datapoint has a DateTime field named t and a Decimal field named value.

## Datatable to Database
We setup a DataTable in memory and fill it with the data we need to insert. 
 
```CSharp
DataTable dt = new DataTable();
dt.Columns.Add("IDTIMESERIES", typeof(int));
dt.Columns.Add("TIMEPOINT", typeof(DateTime));
dt.Columns.Add("VALUE1", typeof(decimal));

int nRowsMaxDt = 0;
//ts being a timeseries object that contains a IEnumberable TimeSeriesData of sp (single points) 
foreach (var sp in ts.TimeSeriesData)
{
	object[] oRowDt = new object[3];
	oRowDt[0] = ts.Id; oRowDt[1] = sp.t; oRowDt[2] = (decimal)sp.value; 
	dt.Rows.Add(oRowDt); nRowsMaxDt = nRowsMaxDt + 1;
}
DumpDataTableToDB("DATATIMESERIES", dt, nRowsMaxDt);
```
The DumpDataTableToDB function is then written on a per database basis. Below you will find my examples for OracleDb, MSSQL and Postgres. 

### Oracle:
Oracle has two sets of drivers, the Unmanaged and the Managed drivers. Your usings will be different per package. 

With ODP.NET Unmanaged (Drivers installed seperately - you will need to add the references manually)
```CSharp
using Oracle.DataAccess.Client;
using Oracle.DataAccess.Types;
```
With ODP.NET Managed (Nuget Installed https://www.nuget.org/packages/Oracle.ManagedDataAccess/ ) 
```CSharp
using Oracle.ManagedDataAccess.Client;
using Oracle.ManagedDataAccess.Types;
```

Use the OracleBulkCopy function with Con being an open connection to the database (OracleConnection). 
```CSharp
public void DumpDataTableToDB(string TableName, DataTable dt, int nCount)
{
	using (OracleBulkCopy obc = new OracleBulkCopy(Con))
	{
		obc.BatchSize = nCount;
		obc.DestinationTableName = TableName;
		obc.WriteToServer(dt);
		obc.Close();
	}
}
```

This method works, but I had issues with Oracle DataGuard. So for Oracle, I do recommend using the Vectorized data insert presented below with Oracle ArrayBind. 

### MS SQL Server
The Microsoft SQL server uses SqlBulkCopy to connect and import the data. Again here the Con represents an open connection to the database. 

```CSharp
using System.Data;
using System.Data.Common;
using System.Data.SqlClient;

public void DumpDataTableToDB(string TableName, DataTable dt, int nCount)
{
using (SqlBulkCopy bulkCopy = new SqlBulkCopy(((SqlConnection)Con).ConnectionString, SqlBulkCopyOptions.KeepNulls))
            {
                foreach (System.Data.DataColumn col in dt.Columns)
                {
                    bulkCopy.ColumnMappings.Add(col.ColumnName, col.ColumnName);
                }
                bulkCopy.DestinationTableName = TableName;
                bulkCopy.WriteToServer(dt);
            }
}
```

### Postgresql
In Postgresql, the important part to note is the COPY ... FROM command. This allows you to provide either a file located on the DB Server or data from STDIN, passed from the client. The Format specifier allows for binary, which we are using here or text and csv. Note, the csv format is very picky, try the text format first if you are doing imports.

As usual the Con represents an open database connection. 

You can see the implementation of this running [in the Jupyiter notebook with C#](https://github.com/ewinnington/noteb/blob/master/PgBulkInsert.ipynb).

```CSharp
using Npgsql; // Nuget https://www.nuget.org/packages/Npgsql 

public void DumpDataTableToDB(string TableName, DataTable dt, int nCount)
{
Dictionary<Type, NpgsqlTypes.NpgsqlDbType> TypeDict = new Dictionary<Type, NpgsqlTypes.NpgsqlDbType>();

            TypeDict.Add(typeof(int), NpgsqlTypes.NpgsqlDbType.Integer);
            TypeDict.Add(typeof(double), NpgsqlTypes.NpgsqlDbType.Double);
            TypeDict.Add(typeof(decimal), NpgsqlTypes.NpgsqlDbType.Numeric);
            TypeDict.Add(typeof(string), NpgsqlTypes.NpgsqlDbType.Varchar);
            TypeDict.Add(typeof(DateTime), NpgsqlTypes.NpgsqlDbType.Timestamp);
            TypeDict.Add(typeof(char[]), NpgsqlTypes.NpgsqlDbType.Varchar);
            TypeDict.Add(typeof(Guid), NpgsqlTypes.NpgsqlDbType.Uuid);


            string sql = "COPY " + TableName + " ( ";
            foreach (System.Data.DataColumn col in dt.Columns)
            {
                sql += (col.ColumnName.ToLower() + ",");
            }
            sql = sql.TrimEnd(',') + ") FROM STDIN (FORMAT BINARY)";
            int nRows = dt.Rows.Count;
            using (var BulkWrite = ((Npgsql.NpgsqlConnection)Con).BeginBinaryImport(sql))
            {
                for (int idRow = 0; idRow < nRows; idRow++)
                {
                    BulkWrite.StartRow();
                    foreach (System.Data.DataColumn col in dt.Columns)
                    {
                        if (dt.Rows[idRow].IsNull(col))
                        {
                            BulkWrite.WriteNull();
                        }
                        else
                        {
                            if (col.DataType == typeof(string) && string.IsNullOrEmpty(dt.Rows[idRow].Field<string>(col)))
                            {
                                BulkWrite.WriteNull();
                            }
                            else
                            {
                                BulkWrite.Write(dt.Rows[idRow][col.Ordinal], TypeDict[col.DataType]);
                            }
                        }
                    }
                    
                }
                BulkWrite.Complete();
            }
        }
    }
```

## Alternative Method for Oracle Array Bind, for Vector Insertion
This is a better method for insertions on Oracle that does not cause issues with dataguard, using arrays of data. The single INSERT command is fed with an array of data. 

```CSharp
int nRowsCount = ts.Count(); 
int[] IDTIMESERIES = new int[nRowsCount];
DateTime[] TIMEPOINT = new DateTime[nRowsCount];
decimal[] VALUE1 = new decimal[nRowsCount];

// loading the arrays with data 
int i = 0;
foreach (var sp in ts.TimeSeriesData)
{
	IDTIMESERIES[i] = ts.Id; TIMEPOINT[i] = sp.t; VALUE1[i] = sp.value;
	i++;
}

if (nRowsCount > 0)
{
	OracleCommand cmdTimeDump = new OracleCommand("Insert into DATATIMESERIES (IDTIMESERIES, TIMEPOINT, VALUE1) VALUES (:1, :2, :3)", OraCon);
	cmdTimeDump.ArrayBindCount = nRowsCount;
	cmdTimeDump.Parameters.Add(new OracleParameter() { OracleDbType = OracleDbType.Int32, Value = IDTIMESERIES });
	cmdTimeDump.Parameters.Add(new OracleParameter() { OracleDbType = OracleDbType.Date, Value = TIMEPOINT });
	cmdTimeDump.Parameters.Add(new OracleParameter() { OracleDbType = OracleDbType.Decimal, Value = VALUE1 });
  
	cmdTimeDump.ExecuteNonQuery();
}
```

# Links

An excellent overview of connections to databases in .net core was published on the [Microsoft devblogs]( https://devblogs.microsoft.com/dotnet/net-core-data-access/). 
