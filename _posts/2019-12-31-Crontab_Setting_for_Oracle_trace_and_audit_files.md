---
layout: post
title: Crontab setting for Oracle trace and audit files
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


The below crontab entries work like so:
 1. Every Sunday at 3:00 AM (OS server time) an archive per group of files is created (Trace, Audit and Dump files),
    the files are transferred to \*TBL
 2. Every Sunday at 4:00 AM the files (Trace, Audit and Dump) are removed from their original location
 3. Every day at 3:00 AM archives older than 30 days are removed from \*TBL

> Trace Backup Location (TBL):
 ```$BACKUP_FOLDER``` - This needs to be either replaced or > > setup prior to editing the crontab

> Make sure the following environment variables are set for the user, which executes the crontab

```$ORACLE_BASE```
```$ORACLE_HOME```
```$BACKUP_FOLDER``` (if you use it)

To open the crontab in edit mode do
```
crontab -e
```

The actual crontab code that does all of the above is

```
#Backs up the files that are to be deleted
0 3 * * sun find $ORACLE_BASE/admin -name "*.aud" | xargs -d "\n" tar -czvf $BACKUP_FOLDER/base_audit_backup_`date +%F_%T`.tar.gz
0 3 * * sun find $ORACLE_BASE/diag/rdbms  -name "*.tr*" | xargs -d "\n" tar -czvf $BACKUP_FOLDER/trace_backup_`date +%F_%T`.tar.gz
0 3 * * sun find $ORACLE_HOME/rdbms/audit -name "*.aud" | xargs -d "\n" tar -czvf $BACKUP_FOLDER/home_aud_backup_`date +%F_%T`.tar.gz

#Removes all the Trace, Audit and Dump files
0 4 * * sun find $ORACLE_BASE/admin -name "*.aud" | xargs rm
0 4 * * sun find $ORACLE_BASE/diag/rdbms  -name "*.tr*" | xargs rm
0 4 * * sun find $ORACLE_HOME/rdbms/audit -name "*.aud" | xargs rm

#Removes archives older than 1 month
0 3 * * * find $BACKUP_FOLDER -name "*.tar.gz" -mtime +30 | xargs rm
```

*Make sure the paths are correct, as the ones here reflect a fairly standard Oracle installation.
