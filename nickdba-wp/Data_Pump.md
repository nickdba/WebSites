# Data Pump

## Prerequisites

In order to export the full content of a database we need few prerequisites.
The export user should have `EXP_FULL_DATABASE` role.

We will use `system` user, so we don't have to mess with the permissions. 
Make sure the system user is unlocked and you know the password.

```sql
alter user system identified by <new_pass> account unlock;
```

We can create a data pump directory, and leave the default alone.

```sql
create directory dpump_dir as '<dir_location>';
```

## Export

Start the database in restrict mode.

```sql
oraenv <db_name> 
sqlplus / as sysdba 
startup force restricted;
exit;
```

Export the database in the newly created location.

```sql
expdp system/<pass> DIRECTORY=dpump_dir FULL=y dumpfile=<db_name>_%u.dmp LOGFILE=exp_<db_name>.log parallel=6
```

## Import

Create a new empty database with dbca.

Prepare the database:

```sql
oraenv <db_name> 
sqlplus / as sysdba 
startup force restricted;
create directory dpump_dir as '<dir_location>';
exit;
```

Import the files:

```sql
impd system/<pass> DIRECTORY=dpump_dir FULL=y dumpfile=<db_name>_%u.dmp LOGFILE=imp_<db_name>.log parallel=6
```
