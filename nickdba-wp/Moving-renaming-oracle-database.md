# Moving or renaming an Oracle database

If you just need to move the location of the oracle database just skip to the moving part.
Otherwise to completely rename the database, you need to do both.

1.[Renaming the database](#renaming-the-database) to rename the database.
2.[Moving the database](#moving-the-database) instructions to change paths.

## Renaming the database

Strictly speaking when you rename a database, you can get away with this part only.
However all the paths and parameters will still keep the old name.
Keep in mind that all the archive logs and rman backups will not be useful anymore as they will contain the old dbid.

First we will put the database in mount mode and disable archive logs. You **will** need to drop all restore points.

```sql
C:\> oraenv <db_name>
C:\> sqlplus sys/<sys_pass> as sysdba

SQL> shutdown immediate
SQL> startup mount
SQL> alter database noarchivelog;
SQL> exit
```

At this point we can also delete `Archivelog` and `Flashback` folders from FRA.

We will use the Oracle nid utility to rename database name.

```bash
C:\> nid target=sys/<sys_pass> DBNAME=<new_db_name>
```

Next we need to replace the oracle service with a new one.

```bash
C:\> oradim -new -sid <new_db_name> -startmode manual -shutmode immediate
C:\> sc delete "OracleService<db_name>"
C:\> sc delete "OracleJobScheduler<db_name>"
C:\> sc delete "OracleVssWriter<db_name>"
```

Rename the password file from `PWD<db_name>.ora` to `PWD<new_db_name>.ora`.
Rename the spfile from `SPFILE<db_name>.ora` to `SPFILE<new_db_name>.ora`.
Update tnsnames.ora and linstener.ora to reflect the new name.

The database is now renamed and you can use it as it is, but all the paths will correspond to the old database.
Keep going to the next section if you want to rename everything.

## Moving the database

1.[Create a pfile](#create-a-pfile)
2.[Create alter database statements](#create-alter-database-statements)
3.[Disable Flashback](#disable-flashback)
4.[Copy database to the new destination](#copy-database-to-the-new-destination )
5.[Post move actions](#post-move-actions)
6.[Remove old copy](#remove-old-copy)

### Create a pfile

Create pfile from spfile

```sql
C:\> oraenv <db_name>
C:\> sqlplus sys/<sys_pass> as sysdba
SQL> create pfile from spfile;
```

Delete all the lines that do not start with an * from the beginning of the file.
Modify the newly created pfile changing all paths and names to mach the desired database.

### Create alter database statements

The following example will change database file destination drive to g:.
You need to change it to suit your needs. When you are happy with the result,
run the selects and copy the results into a new text file.
Alternatively you can just use the selects and manually change each path.

```sql
SQL>  select 'alter database rename file ''' || file_name || ''' to ''g:\' ||substr(file_name, 4) || ''';' from dba_data_files
union all
select 'alter database rename file ''' || file_name || ''' to ''g:\' || substr(file_name, 4) || ''';' from dba_temp_files
union all
select 'alter database rename file ''' || member || ''' to ''g:\' || substr(member, 4) || ''';' from v$logfile
order by 1;
```

### Disable flashback

If you already completed [Renaming the database](#renaming-the-database) part you can skip this section.
You **will** need to drop all restore points.
If you 

```sql
SQL> alter database flashback off;
```

### Copy database to the new destination

Shutdown the database *cleanly*.

```sql
SQL> shutdown immediate
```

Copy database files across. That includes `ORA_DATA` and `FRA`.

### Initialize the database

Start the database with pfile created on the step 1.
Then run the alter database file created at point 2.

```sql
SQL> startup nomount pfile='<pfile.ora>'
SQL> @<alter_database.sql>
```

Check `v$datafiles`, `v$tempfile`, `v$logfile` and `v$controlfile`
to ensure all files are correctly in the new location. Correct the
individual `alter database rename...` commands as appropriate to fix
any problems.

Check startup parameters:

* show parameter recovery\_file\_dest
* show parameter log\_archive\_dest\_1
* show parameter name\_convert

### Post move actions

Restart the database:

```sql
SQL> Alter database open;
```

Create a new spfile from the running pfile:

```sql
SQL> create spfile from pfile='<pfile>.ora';
```

Restart the database to pick up the new spfile:

```sql
SQL> startup force;
```

Enable flashback if needed:

```sql
SQL> alter database flashback on;
```

### Remove old copy

Once **absolutely** certain that all is well, and all files in the
database are being used from the new location, you may remove the old
location and the files within.