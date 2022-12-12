# Database-Backup Project

## Description:
Automate script to perform database backup daily.

## Instruction:

### Install Microsoft SQL Server 2019 on Linux Ubuntu 20.04 Server
- Download the repository configuration

      wget -qO- https://packages.microsoft.com/keys/microsoft.asc | sudo apt-key add -
      sudo add-apt-repository "$(wget -qO- https://packages.microsoft.com/config/ubuntu/20.04/mssql-server-2019.list)"
       
- Install Microsoft SQL Server

      sudo apt-get update
      sudo apt-get install -y mssql-server
       
- Complete the setup of Microsoft SQL Server

      sudo /opt/mssql/bin/mssql-conf setup
      
      o Enter your edition(1-8): 3 	# [Express]
      o Do you accept the license terms? [Yes/No]: Yes
      o Enter the SQL Server system administrator password:  P@ssw0rd
      o Confirm the SQL Server system administrator password: P@ssw0rd

- Verify MSSQL Server Service Is Running

      systemctl status mssql-server
             
- Install MSSQL Server Command Line Tools

      curl https://packages.microsoft.com/keys/microsoft.asc | sudo apt-key add -
      curl https://packages.microsoft.com/config/ubuntu/20.04/prod.list | sudo tee /etc/apt/sources.list.d/msprod.list
      sudo apt-get update
      sudo apt-get -y install mssql-tools unixodbc-dev
      echo 'export PATH="$PATH:/opt/mssql-tools/bin"' >> ~/.bash_profile
      echo 'export PATH="$PATH:/opt/mssql-tools/bin"' >> ~/.bashrc 
      source ~/.bashrc
       
### Create Database and Insert Data
```sql
sqlcmd -S localhost -U sa -P P@ssw0rd
1> CREATE DATABASE Shop;
2> go
1> USE Shop;
2> go
1> CREATE TABLE dbo.Inventory (
2> id INT IDENTITY(1,1) NOT NULL PRIMARY KEY,
3> name NVARCHAR(50),
4> quantity INT
5> );
6> go
1> INSERT INTO dbo.Inventory VALUES('banana',150),('orange',164),('apple',120);
2> go
1> SELECT * FROM dbo.Inventoy;
2> go
```
![sql_query](https://raw.githubusercontent.com/vottri/Database-Backup/main/material/SQL%20query.png)

```sql
1> exit
```
### Backup Script for SQL Server Database

- Create Backup Directory

      sudo mkdir /sql-backup
	  sudo chown mssql:root /sql-backup/
	  sudo chmod 775 /sql-backup/

- Script to Backup Database with Date
  ```
  nano sqlbackupjob.sh
  ```
  ```sh
  #!/bin/bash
  BACKUP_FOLDER='/sql-backup';
  DATABASE='Shop';
  USER='sa';
  PASS='P@ssw0rd';
  FILENAME=$(date +$DATABASE-%d-%m-%Y-%H-%M-%S.bak);

  /opt/mssql-tools/bin/sqlcmd -S localhost -U $USER -P $PASS -Q "backup database $DATABASE to disk='$BACKUP_FOLDER/$FILENAME';"
  ```

- Schedule Crontab to Perform Database Backup Every Day at 00:00

      chmod 775 sqlbackupjob.sh
      crontab -e
      0 0 * * * /bin/sh ~/sqlbackupjob.sh
      
- Check Your Backup Files 

![syslog](https://raw.githubusercontent.com/vottri/Database-Backup/main/material/syslog.png)


       
