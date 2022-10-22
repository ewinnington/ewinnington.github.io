Title: Oracle Getting a list of all columns, data types, lengths of column for every table in your schema
Published: 16/09/2014
Tags: [Migrated, Oracle, Database] 
---

To get all the columns in the schema of an oracle DB, with the appropriate datatype and column length you can use:
```sql
select column_name, data_type, data_length, table_name 
from all_tab_columns where table_name in 
(select TABLE_NAME from user_tables);
```