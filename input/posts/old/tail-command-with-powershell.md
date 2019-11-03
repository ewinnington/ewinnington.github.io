Title: Tail command with powershell
Published: 19/02/2015
Tags: [Migrated, Powershell] 
---

Powershell has now got a command to see the last lines of a file. The syntax is for the last 10 lines:
```powershell
Get-Content _filename _\-Tail 10
```
It also has a -wait command that blocks and shows new entries as they arrive.