# Installing MS Sql server using a config file

As a database admin, you will probably have to install the sql server on multiple machines.
To make sure you have the same parameters and features on all the installation,
it is a good idea to create a configuration ini file. And to do the installtion using that file.

To create the config file, just run the gui installation once and customise it as needed.
Just before you click install on the summary page called "Ready to install" you will find the name and the path of the configuration file.
At this point you can just cancel the installation, go to the path and copy the file elsewhere for modification.

There are few modification you need to do at the very least:

```powershell
IACCEPTPYTHONLICENSETERMS="True"
IACCEPTROPENLICENSETERMS="False"
```

Then we can create a powershell script to do the pre-requesits and run the installation using the config file.

```powershell
Import-Module Servermanager
Add-WindowsFeature NET-Framework-45-Core

New-Item -Path "C:\..\ResultDir" -ItemType "directory"
New-Item -Path "C:\..\WorkingDir" -ItemType "directory"

d:\setup.exe /configurationfile="C:\Install\configurationFile.ini"
```