# Enumeration

```
nmap -v -sV --version-intensity=5 --script imap-capabilities -p T:143 $IP
```

# Interacting w/ service 

```
telnet $IP 143
nc -nv $IP 143

# using TLS
openssl s_client -connect $IP:imaps

# Get TLS version, certificate details & banner
curl -k 'imaps://$IP' --user user:password123 -v
```

# IMAP Commands :

|**Command**|**Description**|
|---|---|
|`1 LOGIN username password`|User's login.|
|`1 LIST "" *`|Lists all directories.|
|`1 CREATE "INBOX"`|Creates a mailbox with a specified name.|
|`1 DELETE "INBOX"`|Deletes a mailbox.|
|`1 RENAME "ToRead" "Important"`|Renames a mailbox.|
|`1 LSUB "" *`|Returns a subset of names from the set of names that the User has declared as being `active` or `subscribed`.|
|`1 SELECT INBOX`|Selects a mailbox so that messages in the mailbox can be accessed.|
|`1 UNSELECT INBOX`|Exits the selected mailbox.|
|`1 FETCH <ID> all`|Retrieves data associated with a message in the mailbox.|
|`1 CLOSE`|Removes all messages with the `Deleted` flag set.|
|`1 LOGOUT`|Closes the connection with the IMAP server.|

[IMAP Commands CheatSheet](https://www.atmail.com/blog/imap-commands/)

# Dangerous Settings :

Configuration mistakes will allow us to read all the emails sent and received.
Some of these configuration include :

| Dangerous settings        | **Description**                                                                           |
| ------------------------- | ----------------------------------------------------------------------------------------- |
| `auth_debug`              | Enables all authentication debug logging.                                                 |
| `auth_debug_passwords`    | This setting adjusts log verbosity, the submitted passwords, and the scheme gets logged.  |
| `auth_verbose`            | Logs unsuccessful authentication attempts and their reasons.                              |
| `auth_verbose_passwords`  | Passwords used for authentication are logged and can also be truncated.                   |
| `auth_anonymous_username` | This specifies the username to be used when logging in with the ANONYMOUS SASL mechanism. |
