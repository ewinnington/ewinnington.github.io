Title: Oracle SQL Developer - Set NLS to give you full date and time
Published: 31/01/2014
Tags: [Migrated, Oracle, Database] 
---

In the SQL sheet for the database, use the following SQL:
```sql
alter session set NLS_DATE_FORMAT = "dd.mm.yyyy hh24:mi:ss";
```