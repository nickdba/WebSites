# Clone a database with RMAN

These steps will focus on refreshing a database with RMAN, either from RMAN backup, or from another database.
Examples shown here are for windows environment, but similar approach can be used in linux as well.

<!--more-->

The refreshed database will be called `db_name` and the source database will be called `source_db`.

The content found here represents a check list rather that an actual tutorial.
Let us begin by defining two scenarios: one scenario in which the we clone to an existing database and one in which we clone to a brand new database.
You can skip directly to the part you want to follow, using the links below:

* [Prepare new database](#prepare-new-database)
* [Prepare existing database](#prepare-existing-database)
* [Start the database](#start-the-database)
* [Clone database from RMAN Backup](#clone-database-from-rman-backup)
* [Clone database from another online database](#clone-database-from-another-online-database)
* [Post clone clean up](#post-clone-clean-up)

<a id="prepare-new-database"></a>

## Prepare new database

Make sure you have oracle database software installed on the machine

Create a service with `oradim` application

```sql
C:\> oradim -new -sid <db_name> -startmode manual -shutmode immediate
```

Create the database folder structure

```sql
C:\> mkdir mnt\ora_data\<db_name>
C:\> mkdir mnt\fast_recovery_area\<db_name>
```

Create a new password file

```sql
C:\> orapwd file=<PWDDB_NAME>.ora password=<db_pass> entries=max_sysdba_users
```

Create a `pfile` called `INITDB_NAME.ora` that will contain only the following.

```sql
db_name=<db_name>
```

Create an entry for `db_name` database in the `tnsnames.ora` file

Create a static entry in the `listener.ora` file as the database will be in `nomount` mode and restart the listener.

```sql
C:\> lsnrctl stop
C:\> lsnrctl start
```

<a id="prepare-existing-database"></a>

## Prepare existing database

First shutdown the database.

```sql
C:\> oraenv <db_name>
C:\> sqlplus sys/<sys_pass> as sysdba
```

Drop the existing database. This will delete all the database files including `spfile` and `control files`:

```sql
SQL> startup force restrict mount
SQL> drop database;
```

Delete archive logs, flashback logs and files residing in `diag\rdbms`.

Check that a static entry for `db_name` database exists in the listener.ora file. If there isn't one, create an entry and restart the listener.

```sql
C:\> lsnrctl stop
C:\> lsnrctl start
```

<a id="start-the-database"></a>

## Start the database

Start the new database in `nomount` mode.

```sql
C:\> oraenv <db_name>
SQL> sqlplus / as sysdba
SQL> startup nomount pfile='?\database\init<db_name>.ora'
SQL> exit
```

<a id="clone-database-from-rman-backup"></a>

## Clone database from RMAN Backup

Connect to RMAN without a target and with auxiliary as the new database.

```sql
C:\> rman auxiliary sys/sys_password@<db_name>
```

The following run block needs to be modified with correct paths and database names. After that can be run directly from RMAN prompt or as a script file with @ sign. (E.g. @clone_script.rman). Backup directory `\\rman_backup\<source_db>\` needs to be accessible, obviously.

```sql
RMAN> run {
  allocate auxiliary channel x1 device type DISK;
  allocate auxiliary channel x2 device type DISK;
  allocate auxiliary channel x3 device type DISK;
  allocate auxiliary channel x4 device type DISK;
  duplicate database <source_db> to <db_name>
  spfile
  set instance_name '<db_name>'
  set service_names '<db_name>'
  set fal_server=''
  set log_archive_config=''
  set log_archive_dest_2=''
  set log_archive_dest_3=''
  set dispatchers '(protocol=tcp) (service=<db_name>xdb)'
  set audit_file_dest 'c:\oracle\admin\<db_name>\adump'
  set db_recovery_file_dest 'c:\fast_recovery_area'
  set dg_broker_start 'false'
  set control_files
  'c:\ora_data\<db_name>\control01.ctl',
  'c:\fast_recovery_area\<db_name>\control02.ctl'
  set db_file_name_convert
  'c:\ora_data\<source_db>',
  'c:\ora_data\<db_name>',
  'c:\fast_recovery_area\<source_db>',
  'c:\fast_recovery_area\<db_name>'
  set log_file_name_convert
  'c:\ora_data\<source_db>',
  'c:\ora_data\<db_name>',
  'c:\fast_recovery_area\<source_db>',
  'c:\fast_recovery_area\<db_name>'

  backup location '\\rman_backup\<source_db>\'
  nofilenamecheck;

  release channel x1;
  release channel x2;
  release channel x3;
  release channel x4;
}
```

Follow the output of rman. After database mounted print, log into the database as `sysdba` and disable block change tracking.

```sql
SQL> alter database disable block change tracking;
SQL> select * from v$block_change_tracking;
```

<a id="clone-database-from-another-online-database"></a>

## Clone database from another online database

The source database needs to be online and to have an `spfile`.
Also it needs to run in `archivelog` mode.

```sql
SQL> show parameter spfile
SQL> select log_mode from v$database;
```

Make sure it is accessible from the machine hosting database `db_name`.

```sql
C:\> tnping <source_db>
```

Connect to RMAN with target as `source_db` and auxiliary as the new database.

```sql
C:\> rman target sys/sys_pass@<source_db> auxiliary sys/sys_pass@<db_name>
```

The following run block needs to be modified with correct paths and database names. After that can be run directly from RMAN prompt or as a script file with @ sign. (E.g. @clone_script.rman).

```sql
RMAN> run {

allocate auxiliary channel x1 device type DISK;
allocate auxiliary channel x2 device type DISK;
allocate auxiliary channel x3 device type DISK;

allocate channel d1 device type DISK;
allocate channel d2 device type DISK;
allocate channel d3 device type DISK;
allocate channel d4 device type DISK;
allocate channel d5 device type DISK;

duplicate target database to <db_name>
from active database
spfile
set instance_name '<db_name>'
set service_names '<db_name>'
set dispatchers '(protocol=tcp) (service=<db_name>xdb)'
set audit_file_dest 'c:\oracle\admin\<db_name>\adump'
set db_recovery_file_dest 'c:\fast_recovery_area'
set dg_broker_start 'false'
set control_files
'c:\ora_data\<db_name>\control01.ctl',
'c:\fast_recovery_area\<db_name>\control02.ctl'
set db_file_name_convert
'c:\ora_data\<source_db>',
'c:\ora_data\<db_name>',
'c:\fast_recovery_area\<source_db>',
'c:\fast_recovery_area\<db_name>'	 
set log_file_name_convert
'c:\ora_data\<source_db>',
'c:\ora_data\<db_name>',
'c:\fast_recovery_area\<source_db>',
'c:\fast_recovery_area\<db_name>'
nofilenamecheck;

release channel x1;
release channel x2;
release channel x3;
release channel d1;
release channel d2;
release channel d3;
release channel d4;
release channel d5;
}
```

<a id="post-clone-clean-up"></a>

## Post clone clean up

After RMAN finishes, perform post restore cleanups if necessary.

* Disable jobs
* Enable block change tracking
* Change passwords
* Drop/update database links
* Run configuration scripts
* Configure rman backups
