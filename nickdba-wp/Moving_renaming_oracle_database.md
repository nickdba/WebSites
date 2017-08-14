# Moving or renaming an Oracle database

# Renaming Oracle database has two parts:

1. [Use nid utility](#use-nid-utility) to rename the database.
2. [Moving Oracle database](#moving-oracle-database) instructions to change paths.

## Use nid utility

You can use the Oracle nid utility to rename database name. If you don't need to rename the database, just skip ahead to next section.

```sql
C:\> nid target=sys/<sys_pass>@<db_name> DBNAME=<new_db_name>
```

This command will rename the database, but all the paths will contain the old db name.

# Moving Oracle database

1. [Create a pfile](#create-a-pfile)
2. [Create alter database statements](#create-alter-database-statements)
3. [Disable Flashback](#disable-flashback)
4. [Copy database to the new destination](#copy-database-to-the-new-destination )
5. [Post move actions](#post-move-actions)
6. [Remove old copy](#remove-old-copy)

## Create a pfile

Create pfile from spfile

```sql
SQL> oraenv <db_name>
SQL> sqlplus sys/<sys_pass> as sysdba
SQL> create pfile='<pfile.ora>' from spfile; 
```

Delete all the lines that do not start with an * from the beginning of the file.
Modify the newly created pfile changing all paths and names to mach the desired database.

## Create alter database statements

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

## Disable flashback

 You **will** need to drop all restore points.

```sql
SQL> alter database flashback off;
```

## Copy database to the new destination 

Shutdown the database *cleanly*.

```sql
SQL> shutdown immediate
```

Copy database files across. That includes `ORA_DATA` and `FRA`.

If the new destination is on a new machine you have 3 more steps:

* create a service with `oradim`
* copy %ora_home%/database content like spfile, password file etc.
* add the database to tnsnames.ora and to linstener.ora

## Initialize the database.

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

*   show parameter recovery\_file\_dest
*   show parameter log\_archive\_dest\_1
*   show parameter name\_convert

## Post move actions

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

Enable flashback:

```sql
SQL> alter database flashback on;
```

## Remove old copy

Once **absolutely** certain that all is well, and all files in the
database are being used from the new location, you may remove the old
location and the files within.