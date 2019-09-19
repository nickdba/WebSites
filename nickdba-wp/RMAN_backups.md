# RMAN Backups

I added few RMAN commands for easier future reference.

<!--more-->

Table of Contents

1. [Introduction](#Introduction)
2. [Backup commands](#backup-commands)
3. [Restore commands](#restore-commands)

<a id="introduction"></a>

## Introduction

Few things to consider before backing up a database.

### Backup set or image copy

A backup set is an RMAN-specific format, whereas an image copy is a bit-for-bit copy of a file. By default, RMAN creates backup sets.
An image copy doesn't depend on RMAN for restoration.

### Consistent or Inconsistent RMAN Backups

For a consistent backup, the database must shutdown gracefully and be mounted.
Inconsistent backups however, have better database availability.

### Compressed backup sets

Disk space vs processor power.

```sql
RMAN> CONFIGURE DEVICE TYPE DISK PARALLELISM 2 BACKUP TYPE TO COMPRESSED BACKUPSET;
```

### Full backup vs incremental backups

By default, RMAN makes full backups.
Incremental backups can be differential or cumulative.
A differential incremental backup, which backs up all blocks changed after the most recent backup at level 1 or 0.
A cumulative incremental backup, which backs up all blocks changed after the most recent backup at level 0.

### Autobackup Controlfile 

You need CONTROLFILE AUTOBACKUP to be ON in order to restore controlfile from backups.

```sql
RMAN> CONFIGURE CONTROLFILE AUTOBACKUP ON;
```

<a id="backup-commands"></a>

## Backup commands

### Consistent full backup

Shutdown the database cleanly and start it in mount mode.
Make sure archive logs are enabled.

```sql
  \> sqlplus sys/syspass as sysdba
SQL> shutdown immediate
SQL> startup mount
SQL> Select log_mode From v$database;
SQL> alter database archivelog;
SQL> exit
```

Back up the database and archive logs using RMAN.
We will use the default options:
Backup type is backupset, device type is disk and output is the FRA with automatic generated name tag. (E.g. TAG20180104T102400)

```sql
   \> rman target sys/syspass@dbname
RMAN> backup database;
RMAN> backup archivelog all;
```

If by any chance backup files are missing.

```sql
RMAN> crosscheck archivelog all;
RMAN> list expired archivelog all;
RMAN> delete expired archivelog all;
RMAN> delete obsolete;
RMAN> backup archivelog all;
```

### Inconsistent differential incremental backup

First we need to take a level 0 backup which is similar to a full backup.
We will also use the compress option just for fun.

```sql
RMAN> crosscheck archivelog all;
RMAN> backup as compressed backupset incremental level 0 database plus archivelog;
```

At a latter time we can backup only the changes.

```sql
RMAN> crosscheck archivelog all;
RMAN> backup as compressed backupset incremental level 1 database plus archivelog;
```

<a id="restore-commands"></a>

## Restore commands

Since we are not using a recovery catalog we will restore control file from backup.

```sql
RMAN> set dbid 12345;
RMAN> startup nomount;
RMAN> restore controlfile from autobackup;
```

Next we will mount the database, restore the database.

```sql
RMAN> startup mount;
RMAN> restore database;
RMAN> exit;
```

Connect as sys in sqlplus set autorecovery as on

```sql
SQLPLUS> SET AUTORECOVERY ON
```

Connect to RMAN again and recover the database.
If and only if you get `media recovery complete` you can reset the logs.

```sql
RMAN> recover database;
RMAN> alter database open resetlogs;
```
