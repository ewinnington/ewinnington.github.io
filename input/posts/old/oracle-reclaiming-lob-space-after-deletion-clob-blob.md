Title: Oracle - Reclaiming LOB space after deletion (CLOB, BLOB)
Published: 03/02/2014
Tags: [Migrated, Oracle] 
---

To reclaim space after a deleting LOBs, use the following commands. XXXXX is my tablename, data is my column name for the lob, in pink the size of the table.

1) Check size of the table XXXXX
```
SELECT table_name, column_name, segment_name, a.bytes
FROM dba_segments a JOIN dba_lobs b
USING (owner, segment_name)
WHERE b.table_name = 'XXXXX';
```
```
>> XXXXX DATA SYS_LOB0000025315C00007$$ 1073741824
```
2) Delete, in my case, I just clean up all files in my table.
```
DELETE FROM XXXXX;
COMMIT;
```
```
>> 183 rows deleted.
>> committed.
```
3) Ask Oracle to shrink the tablespace used by the LOB column ‘data’. Have a coffee break.
```
ALTER TABLE XXXXX MODIFY LOB (data) (SHRINK SPACE);
```
```
>> table XXXXX altered.
```
4) Check size again:
```
SELECT table_name, column_name, segment_name, a.bytes
FROM dba_segments a JOIN dba_lobs b
USING (owner, segment_name)
WHERE b.table_name = 'XXXXX';
```
```
>> XXXXX DATA SYS\_LOB0000025315C00007$$ 65536
```