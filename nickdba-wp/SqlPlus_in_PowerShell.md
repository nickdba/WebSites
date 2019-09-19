# SqlPlus in Powershell

 As a dba inevitably, at some point in time, you will need to automate tasks.
 If you are running the oracle database in windows server environment, the best tool to automate things is Powershell.

There are few ways to call a `Sqlplus` command in `Powershell`, they all have different levels of functionality and flexibility.

If you want to provide more commands to `Sqplus`, you can create a file and execute it like so:

```sql
& 'sqlplus' '<user>/<pass>@db_name' '@my_script.sql'
```

`my_script.sql` needs to have an `exit;` command in the end, other wise you will get `SQL>` prompt rather than continue.

While this method is good, if the content of `my_script.sql` changes often, it is a hustle to modify it.

One method that I use, is to create a `here-string` and then pass it to `Sqlplus`.

```sql
 @'
spool test.lst
Select * From dual;
spool off
exit
'@ | sqlplus <user>/<pass>@db_name
```
