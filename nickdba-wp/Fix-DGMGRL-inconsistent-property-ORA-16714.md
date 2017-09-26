# Fix DGMGRL inconsistent property (ORA-16714)

Doing some checks on DGMGRL after I created a standby database, I got (ORA-16714) error.

```sql
DGMGRL> show database "db_name"

Database - <db_name>
Role:            PHYSICAL STANDBY
Intended State:  APPLY-ON
Transport Lag:   0 seconds (computed 0 seconds ago)
Apply Lag:       0 seconds (computed 0 seconds ago)
Apply Rate:      401.00 KByte/s
Real Time Query: OFF
Instance(s):
  <db_name>
    Warning: ORA-16714: the value of property DbFileNameConvert is inconsistent with the database setting
Database Status:
WARNING
```

First we want to see the differences.

```sql
DGMGRL> show database "db_name" InconsistentProperties
INCONSISTENT PROPERTIES
  INSTANCE_NAME  PROPERTY_NAME      MEMORY_VALUE   SPFILE_VALUE   BROKER_VALUE
  db_name        DbFileNameConvert  <some_values>  <some_values>  <other_values>
```

If we need to modify the database, the name of the parameter will be changed with underscores. (E.g. `DbFileNameConvert` - `db_file_name_convert`)

```sql
SQL> show parameter db_file_name_convert;
SQL> alter system set db_file_name_convert='<primary_oradata>','<standby_oradata>','<primary_fra>','<standby_fra>' scope=spfile;
```

Please be careful with the quotes and the comas, otherwise you might need to read [Fix a broken spfile (ORA-01678)](./Fix-a-broken-spfile-ORA-01678) article.
Restart the database.

If we need to modify the broker.

```sql
DGMGRL> edit database "db_name" set property dbFileNameConvert='<primary_oradata>, <standby_oradata>, <primary_fra>, <standby_fra>';
```

Oracle likes to make things confusing, by not being consistent.
Database parameter value is a set of strings separated by comas. However the broker property is a string containing values separated by comas.

After all these modifications things are fine.

```sql
DGMGRL> show database "db_name"

Database - <db_name>
Role:            PHYSICAL STANDBY
Intended State:  APPLY-ON
Transport Lag:   0 seconds (computed 0 seconds ago)
Apply Lag:       0 seconds (computed 0 seconds ago)
Apply Rate:      401.00 KByte/s
Real Time Query: OFF
Instance(s):
  <db_name>

Database Status:
SUCCESS
```

We like success! :)