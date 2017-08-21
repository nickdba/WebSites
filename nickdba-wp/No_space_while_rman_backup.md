# No space while RMAN backup

While running the `RMAN` backup, the script fails saying there is not enough space on the disk.
<!--more-->

```sql
ORA-27072: File I/O error
OSD-04008: WriteFile() failure, unable to write to file
O/S-Error: (OS 112) There is not enough space on the disk.
```

## Investigation

We first check what is the `RMAN` configuration for this database

```sql
RMAN> show all;
```

and we are manly interested in `RETENTION POLICY` and `ARCHIVELOG DELETION POLICY`

```sql
CONFIGURE RETENTION POLICY TO RECOVERY WINDOW OF 90 DAYS;
CONFIGURE ARCHIVELOG DELETION POLICY TO APPLIED ON ALL STANDBY BACKED UP 2 TIMES TO DISK;
```

Although the retention window is 90 days, checking all backups on the disk shows lots of backups
beyond that date.

```sql
RMAN> list backup summary;

Key     TY LV S Device Type Completion Time     #Pieces #Copies Compressed Tag
------- -- -- - ----------- ------------------- ------- ------- ---------- ---
187321  B  A  A DISK        2015/08/02 05:06:24 1       1       YES        TAG20150802T030506
187322  B  A  A DISK        2015/08/02 05:06:26 1       1       YES        TAG20150802T030506
187323  B  A  A DISK        2015/08/02 05:06:30 1       1       YES        TAG20150802T030506
187324  B  A  A DISK        2015/08/02 05:06:49 1       1       YES        TAG20150802T030506
187325  B  A  A DISK        2015/08/02 05:08:22 1       1       YES        TAG20150802T030506
187326  B  A  A DISK        2015/08/02 05:08:35 1       1       YES        TAG20150802T030506
```

## Solution

The solution is to delete obsolete and expired Oracle RMAN backups.
But before that we will do a crosscheck of backups. Crosscheck verify the actual files against the rman catalog, and updates the catalog.

```sql
RMAN> crosscheck backup;
...
Crosschecked 256 objects
```

`Delete obsolete;` command will apply the retention window and delete all the obsolete backups and archive logs.
If you want to modify the retention window of the delete on the fly:

```sql
RMAN> delete obsolete recovery window of 120 days;

Type: YES
```

All the backups and archive logs that crosscheck didn't match in rman catalog will be market expired. To delete the expired backups;

```sql
RMAN> delete expired backup;
```

>Big thanks to Mr. Ramesh Natarajan for his article [Delete oracle rman backup](http://www.thegeekstuff.com/2015/01/delete-oracle-rman-backup)
>Him and the Official Oracle Manual provided big help in resolving this issue.