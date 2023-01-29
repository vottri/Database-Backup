# Database-Backup Project

## Description of task:

Automate script to perform database backup daily.

## Instruction steps:

### Set up a Linux virtual machine.

This lab was done by renting virtual machines on Microsoft Azure Cloud. You can create an Azure free account for practicing this lab.

Visit the Azure platform page at https://portal.azure.com/

Sign into your account. A successful sign in takes you to the Azure Dashboard page.

Click on **Virtual Machines** to create a Linux VM for your lab.

![a1](https://raw.githubusercontent.com/vottri/Database-Backup/main/material/a1.png)

Choose **Create** > **Azure virtual machine**.

![a2](https://raw.githubusercontent.com/vottri/Database-Backup/main/material/a2.png)

We are going to create Linux Virtual Machine on Azure with Image of Ubuntu Server 20.04 LTS.

You can choose your subscription which you have when you create your account. If you have existing resource group then you can select that or create a new resource group if you like.

I am going to name this VM **u2svr**. You can choose the name of VM as you want. Next, select a region, I am going with **South Central US**. Operation System for VM (image) is going to be **Ubuntu Server 20.04 LTS – Gen2**.

For the size of Virtual Machine, we don’t need a large machine for this lab, that why I have chosen **Standard_B2s instance which has 2 vCPU(s) and 4GB RAM**.

![a3](https://raw.githubusercontent.com/vottri/Database-Backup/main/material/a3.png)

Provide Username & Password for the logon user.

Choose **Allow selected ports** and in **Select inbound ports** section, we are going to select port 80 (HTTP) and port 22 (SSH).

![a4](https://raw.githubusercontent.com/vottri/Database-Backup/main/material/a4.png)

Choose **Standard SSD** disk type.

![a5](https://raw.githubusercontent.com/vottri/Database-Backup/main/material/a5.png)

You can leave everything as default here. 

![a6](https://raw.githubusercontent.com/vottri/Database-Backup/main/material/a6.png)

When you are ready, click **Review + create**. When you pass the validation process, click **Create**.

![a7](https://raw.githubusercontent.com/vottri/Database-Backup/main/material/a7.png)

It will take some time for deployment. When your VM is ready, click **Go to resource**.

![a8-1](https://raw.githubusercontent.com/vottri/Database-Backup/main/material/a8-1.png)

![vm1](https://raw.githubusercontent.com/vottri/Database-Backup/main/material/vm1.png)

Use **PuTTY** for connecting to the Linux Virtual Machine. Enter your Linux machine's public IP address in Host Name and Port will be 22. Click on **Open** button to connect.

![vm2](https://raw.githubusercontent.com/vottri/Database-Backup/main/material/vm2.png)

You are now inside the Linux Virtual machine.

![vm3](https://raw.githubusercontent.com/vottri/Database-Backup/main/material/vm3.png)

Check for working Internet connection.

Update your system

```sh
sudo apt-get update
```

Install some required packages:

```sh
sudo apt-get -y install wget apt-transport-https
```

### Install Microsoft SQL Server 2019 on Linux Ubuntu 20.04 Server

- **Download the repository configuration**

```sh
wget -qO- https://packages.microsoft.com/keys/microsoft.asc | sudo apt-key add -
sudo add-apt-repository "$(wget -qO- https://packages.microsoft.com/config/ubuntu/20.04/mssql-server-2019.list)"
```

- **Install Microsoft SQL Server**

```sh
sudo apt-get update
sudo apt-get install -y mssql-server
```

- **Complete the setup of Microsoft SQL Server**

```sh
sudo /opt/mssql/bin/mssql-conf setup
```

```
o Enter your edition(1-8): 3 	# [Express]
o Do you accept the license terms? [Yes/No]: Yes
o Enter the SQL Server system administrator password:  P@ssw0rd
o Confirm the SQL Server system administrator password: P@ssw0rd
```

![mssqlsetup](https://raw.githubusercontent.com/vottri/Database-Backup/main/material/mssqlsetup.png)

- **Verify MSSQL Server Service Is Running**

```sh
systemctl status mssql-server
```             

![mssqlstatus1](https://raw.githubusercontent.com/vottri/Database-Backup/main/material/mssqlstatus1.png)

- **Install MSSQL Server Command Line Tools**

```sh
curl https://packages.microsoft.com/keys/microsoft.asc | sudo apt-key add -
curl https://packages.microsoft.com/config/ubuntu/20.04/prod.list | sudo tee /etc/apt/sources.list.d/msprod.list
sudo apt-get update
sudo apt-get -y install mssql-tools unixodbc-dev
echo 'export PATH="$PATH:/opt/mssql-tools/bin"' >> ~/.bash_profile
echo 'export PATH="$PATH:/opt/mssql-tools/bin"' >> ~/.bashrc 
source ~/.bashrc
```

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

- **Create Backup Directory**

```sh
sudo mkdir /sql-backup
sudo chown mssql:root /sql-backup/
sudo chmod 775 /sql-backup/
```

- **Script to Backup Database with Date**

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

```sh
chmod 775 sqlbackupjob.sh
```

- **Schedule Crontab to Perform Database Backup Everyday at 00:00**

```sh
crontab -e
0 0 * * * /bin/sh ~/sqlbackupjob.sh
```

![cron1](https://raw.githubusercontent.com/vottri/Database-Backup/main/material/cron1.png)

![cron2](https://raw.githubusercontent.com/vottri/Database-Backup/main/material/cron2.png)

- **Check Your Backup Files**

![syslog](https://raw.githubusercontent.com/vottri/Database-Backup/main/material/syslog.png)


       
