# Flashback

I have added few flashback commands for easy future reference.

<!--more-->

Table of Contents
1. [Checks](#checks)
2. [Set up](#set-up)
3. [Flashback commands](#flashback-commands)
4. [Drop and Disable](#drop-and-disable)

<a id="checks"></a>

## Checks 
Check if the `archivelog` mode is enabled, `flashback_on` and the log path is set. The possible values for `log_mode` are 'archivelog' and 'noarchivelog'. The possible values for `flashback_on` are 'Yes' and 'No'. 

```sql
SQL> select log_mode, flashback_on from v$database;
SQL> select value from v$parameter Where name='log_archive_dest_1';
```

To check flashback retention buffer in days and to the time you can actually go back to:

```sql
SQL> select name, (value/60/24) as days from v$parameter Where name = 'db_flashback_retention_target';
SQL> select * from v$flashback_database_log;
```

To check all the `restore points` in the database:

```sql
SQL> select * from v$restore_point;
```

<a id="set-up"></a>

## Set up

If log_mode is `noarchivelog`, we need to put the database in `mount` mode and enable `archivelog` mode. Then we can enable `flashback` mode.

```sql
SQL> shutdown immediate
SQL> startup mount
SQL> alter database archivelog;
SQL> alter database flashback on;
SQL> alter database open;
```

To create a restore point:

```sql
SQL> create restore point CLEAN_DB guarantee flashback database;
```

<a id="flashback-commands"></a>

## Flashback commands

Flashback to restore point:

```sql
SQL> shutdown immediate
SQL> startup mount
SQL> flashback database to restore point CLEAN_DB;
SQL> alter database open resetlogs;
```

To flashback to just before one hour ago:

```sql
SQL> shutdown immediate
SQL> startup mount
SQL> flashback database to before timestamp SYSDATE-1/24;
SQL> alter database open resetlogs;
```

If you dropped a table by mistake and did not purge, to restore it:

```sql
SQL> flashback table table_name to before drop;
```

To flashback a table to a point in time:

```sql
SQL> alter table table_name enable row movement;
SQL> flashback table table_name to timestamp to_timestamp('2016-05-08 13:38:00', 'YYYY-MM-DD HH24:MI:SS');
```

Please note that this will act like an insert not really like a restore.

<a id="drop-and-disable"></a>

## Drop and Disable

Drop a restore point.

```sql
SQL> drop restore point CLEAN_DB;
```

Disable `flashback` and `archivelogs`.

```sql
SQL> shutdown immediate
SQL> startup mount
SQL> alter database flashback off;
SQL> alter database noarchivelog;
SQL> alter database open;
```
