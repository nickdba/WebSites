# Fix a broken spfile (ORA-01678)

I managed to break the spfile by modifying `log_file_name_convert` with wrong number of quotes. This parameter doesn't allow `scope=both`, so by using `scope=spfile` and restarting the database, all hell broke loose:
<!--more-->

```sql
SQL> startup
ORA-01678: parameter log_file_name_convert must be pairs 
of pattern and replacement strings
```

Creating a `pfile` from `spfile` didn't work at this stage either.

Good thing it was not pre-production database, or anything like that... (it was pre-production, don't tell anyone :).

By digging around the web, I found this page: <a href="http://qdosmsq.dunbar-it.co.uk/blog/2015/04/how-to-fix-a-broken-asm-spfile-held-within-asm/"> how-to-fix-a-broken-asm-spfile-held-within-asm </a>, created by, surprise, a colleague of mine: Norman Dunbar. Have a reading for more info about the topic. I personally liked the method 2, which I will be describe below.

I created a `pfile` that called the broken spfile and overwrote the wrong parameter at the end.

```sql
*.spfile="oracle_home\database\SPFILEDBNAME.ORA"
*.log_file_name_convert='oradata\db1', 'oradata\db2', 
'fra\db1' 'fra\db2'
```

Started the database using the newly created `pfile`.

```sql
SQL> startup pfile='?\initdbname.ora';
```

Fixed the broken spfile by updating the parameter with the correct values.

```sql
SQL> alter system set log_file_name_convert='oradata\db1', 
'oradata\db2', 'fra\db1' 'fra\db2' scope=spfile;
```

Shutdown and startup. Everything was peachy again.