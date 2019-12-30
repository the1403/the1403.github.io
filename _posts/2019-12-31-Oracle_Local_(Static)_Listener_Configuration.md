---
layout: post
title: Local (Static) Listener Configuration
tags:
- listener.ora
- CDB
- PDB
- Oracle
---

<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/4.7.0/css/font-awesome.min.css">
<script type="text/javascript" src="https://cdnjs.cloudflare.com/ajax/libs/clipboard.js/1.5.16/clipboard.min.js"></script>
<script>
var clipboard = new Clipboard('.btn');
clipboard.on('success', function(e) { console.log(e); });
clipboard.on('error', function(e) { console.log(e); });
</script>

_There are occasions, in which when a Database gets shut down, the default listener automatically unregisters the Databases from it._
_In Oracle 12.2. an on, when a shutdown is issued (on Container level),_
_the Database closes all of the Pluggable Databases and then shuts down the Container Database._
_This process (in the case of the default listener) unregisters both the Container Database and_
_all of the Pluggable Databases in it from the listener and thus is making them inaccessible._
_This is problematic, as this essentially "cuts" the SQLNET link to the Database (unless you have a ssh access to the machine),_
_in case someone wants to administer it through it._
_This problem is solved by configuring a local listener with some static entries in it for the corresponding Databases_
_(Container or Pluggable) to remain "open" to SQLNET connections._

In order to configure a local/static listener, the following steps are necessary:

1. Stop the currently running listener
2. Create a listener.ora file (and place it under $ORACLE_HOME/network/admin)
3. Set the Database parameter local_listener on Container and Pluggable Database level
4. Restart the listener

Stop the currently running listener
```
lsnrctl stop
```

Create the listener.ora file
```
$ vi $ORACLE_HOME/network/admin/listener.ora

# Note: The listener is called LISETNER, which replaced the default listener by the same name
#       This is done, so that you do not have to specify a listener name when starting, stopping or reloading it

LISTENER=
  (DESCRIPTION=
    (ADDRESS_LIST=
      (ADDRESS=(PROTOCOL=tcp)(HOST=<<host_ip_address_or_name>>)(PORT=<<host_port_number>>))
        ))
#Might not be needed, because of the EM
SID_LIST_LISTENER=
  (SID_LIST=
    (SID_DESC=
      (GLOBAL_DBNAME=<<cdb_name>>)
      (ORACLE_HOME=<<oracle_home_location>>)
      (SID_NAME=<<cdb_name>>))
    (SID_DESC=
      (GLOBAL_DBNAME=<<pdb_name>>)
      (ORACLE_HOME=<<oracle_home_location>>)
      (SID_NAME=<<pdb_name>>))
  )
CRS_NOTIFICATION_LISTENER=off
SAVE_CONFIG_ON_STOP_LISTENER=true
SUBSCRIBE_FOR_NODE_DOWN_EVENT_LISTENER=off ## Do not unregister the Database on shutdown
```

Set the local_listener parameter
```
$ sqlplus / as sysdba
--This is on Container level
SQL> alter system set local_listener = '(ADDRESS=(PROTOCOL=tcp)(HOST=<<host_ip_address_or_name>>)(PORT=<<host_port_number>>))';
SQL> alter session set container = <<database_name>>;
--This is on Pluggable Database level; do this for all the Pluggable Databases that are to be registered with this listener
SQL> alter system set local_listener='(ADDRESS=(PROTOCOL=tcp)(HOST=<<host_ip_address_or_name>>)(PORT=<<host_port_number>>))';
```

Start the LISTENER again
```
$ lsnrctl start
```
