# Remove Oracle Databases and Software from Windows

This is a checklist that you can use while removing oracle databases and agents from a windows machine.

## Remove databases

Remove databases using `dbca`.
Remove all the software that you can using universal installer.
Go to the %orahome%/deinstall/ and run deinstall.bat.

## Delete directories

If you have issues deleting oracle installation directories.
In particular `oci.dll`, stop the following services, if they exist:

- COM+ System Application
- VMware Tools Service
- Distributed Transaction Coordinator
- TLM server manager

Delete oracle installation directories.
Start again the services.

## Delete registries

- HKEY_CURRENT_USER\SOFTWARE\ORACLE
- HKEY_LOCAL_MACHINE\SOFTWARE\ORACLE

## Delete Environment variables

From the `Path` environment variable, delete all the paths containing the word oracle.