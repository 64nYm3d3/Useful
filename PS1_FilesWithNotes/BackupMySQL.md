Via

https://cyanghost.com/scripts/Backup-MSSQL-Databases.ps1

~~~
# This PowerShell script will perform a full backup of all defined MSSQL databases to a BAK file and will retain them for one month.
# Use Task Scheduler to have this script run daily. The account you run this under will require the proper permission to perform backups in SQL server.
# Last updated: 7/20/2018

# Define date parameters.
$timestamp = Get-Date -Format yyyy-MM-dd

# Set SQL Server\SQL Database.
$sqldatabase = "ComputerName\InstanceName"

# Set backup path.
$backuppath = "E:\Backups"

# Start log.
Start-Transcript -Path $backuppath\MSSQL_Backup_Log_$timestamp.log -Append

# Create backup folder.
Write-Host "[1] Creating backup folder..."
New-Item -Path "$backuppath" -Name "$timestamp" -ItemType "directory"

# Start backing up SQL databases. Where DATABASENAME, replace with your actual database name.
Write-Host "[2] Backing up database(s)..."

SQLCMD.EXE -E -S $sqldatabase -Q "BACKUP DATABASE DATABASENAME TO DISK='$backuppath\$timestamp\DATABASENAME_$timestamp.bak' WITH FORMAT"

# Verify backed up SQL databases. Where DATABASENAME, replace with your actual database name.
Write-Host "[3] Verifying backup file(s)..."

SQLCMD.EXE -E -S $sqldatabase -Q "RESTORE VERIFYONLY FROM DISK = '$backuppath\$timestamp\DATABASENAME_$timestamp.bak'"

# Remove folders and files older than 30 days.
Write-Host "[4] Removing old files/folders..."

# Minimum age of files/folders to delete.
$limit = (Get-Date).AddDays(-30)

# Delete files/folders older than specified time.
Get-ChildItem -Path $backuppath -Recurse -Force | Where-Object { !$_.PSIsContainer -and $_.CreationTime -lt $limit } | Remove-Item -Force

# Delete empty directories.
Get-ChildItem -Path $backuppath -Recurse -Force | Where-Object { $_.PSIsContainer -and (Get-ChildItem -Path $_.FullName -Recurse -Force | Where-Object { !$_.PSIsContainer }) -eq $null } | Remove-Item -Force -Recurse

# End of script.
Write-Host "[5] Backup complete."

Write-Host "Exiting..."
~~~
