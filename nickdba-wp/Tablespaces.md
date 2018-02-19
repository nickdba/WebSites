# TableSpaces

Easy reference tablespace commands have been added below. Temp tablespace and a troubleshooting sections is also included.


## Temp Tablespace

Adding a file to the temp tablespace.

```sql
Select * From dba_temp_files;
alter tablespace temp add tempfile 'oradata_path:\temp01.dbf' size 100m autoextend on next 100m maxsize unlimited;
```

## Normal Tablespace

Adding a file to a tablespace.

```sql
Select * From dba_data_files Where tablespace_name='TBS_NAME';

alter tablespace tbs_name add datafile 'oradata_path:\tbs_name01.dbf' size 100m autoextend on next 100m maxsize unlimited;
```

Resizing a datafile

```sql
alter database datafile 'oradata_path:\tbs_name01.dbf' resize 500m;
```

Resizing datafile maxsize

```sql
alter database datafile 'oradata_path:\tbs_name01.dbf' autoextend on maxsize 8G;
```

## Troubleshooting

If oracle cannot read from temp data file and you encounter the following error

```sql
ora-01187: cannot read from file because it failed verification tests
ora-01110: data file 201: ‘oradata_path:\temp01.dbf’
```

This is a way to recreate the file:

```sql
alter tablespace temp add tempfile 'oradata_path:\temp02.dbf' size 10m;
alter tablespace temp drop tempfile 'oradata_path:\temp01.dbf';
alter tablespace temp add tempfile 'oradata_path:\temp01.dbf' reuse;
alter tablespace temp drop tempfile 'oradata_path:\temp02.dbf';
```

## Notes

Unlimited, depending of your block size can actually mean 32GB.
On next is the next extend block. The size of extend block should  depend on how fast is the tablespace expected to increase.
If a tablespace is used by a session, it will become offline rather than being deleted.