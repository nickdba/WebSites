# Fix ORA-01000 maximumopen cursors exceeded

This is caused by a session opening more than allowed number of cursors. Most likely due to erroneous code that does not free memory up. One solution si to kill the problematic session.

First we need to check the maximum allowed number of cursors.

```sql
sqlplus sys as sysdba 
show parameter open_cursors
```

By default you should have something like 300.
Next we will find the culprit sid.

```sql
select count(*), sid From v$open_cursor group by sid order by 1 desc;
```

You should look for the one seed that reached maximum open cursors.
In our case 300 open cursors. Note the sid number down we will used it in the next query.
In our example the sid number is 1009.

```sql
select * From v$session Where sid=1009;
alter system kill session 'sid,serial#';
```