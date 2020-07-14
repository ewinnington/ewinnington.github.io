Title: Checking for liveness on databases for health checks
Published: 14/07/2020
Tags: [Oracle, MsSql, Postgresql, SQLite, MySql, MariaDb] 
---

When you just want to check a DB is reachable from your api or code, a health check is used. For the following DBs the simplest query is: 

- Oracle: ```SELECT 1 FROM dual```
- Postgresql: ```SELECT 1```
- SQLite: ```SELECT 1```
- Mysql / MariaDb: ```SELECT 1```
- Microsoft SQL-Server: ```SELECT 1```

There is really an odd one on the list.
