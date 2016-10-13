# backup
Script to backup a server to a remote backup server

This script can be used to setup automatic rotating snapshots of your primary server
to a remote backup server or a second disk on the same server. Actually you don't
have to backup the whole server and can backup any directory on the server. But
if you start the backup from / you will backup the whole server.

## Backup procedure
Lets say that domain.com is the server that we want to backup and that
backup.com is the server where the backup will be made. The backup.com server
must have very large disks; at least a few times larger than the domain.com server.

We have backup.com use rsync to pull all the data domain.com.
This requires backup.com to login as root on domain.com, so that everything
will be readable. Login can be automated using key based login. Ideally
the domain.com server should be setup to only allow key based login and
not accept remote password based login for root.
The `lo` script (from my [`lo` repo](/osyed/lo)) can be used to setup key based login.
```
lo domain = root@domain.com
lo domain auto
```

On the backup.com server you will need to place a copy of the
backitup script. 


The backup script can be run as follows:
```
..../backitup root@domain.com:/ /backup/domain.com 3
```

The backup script will put
the most current backup in a directory called '1' under the directory given
as the second argument. So in this case the most current backup would be in
`/backup/domain.com/1`.

If the '1' directory already exists, it will be rename to '2' and the '2' directory
will be copied back to '1' using hard links reduce disk space usage.

The backup script does an rsync command to the '1' directory so that only changes will need
to be transfered over the network.

Regular backups can be scheduled using cron. For example:
```
1 * * * * /root/cron/backitup rooter@domain.com:/ /backup/domain.com/hour 50 >> /backup/domain.com/log.hour
9 1 * * * /root/cron/backitup /backup/domain.com/hour/1 /backup/domain.com/day 10 >> /backup/domain.com/log.day
1 2 * * 1 /root/cron/backitup /backup/domain.com/day/1 /backup/domain.com/week 10 >> /backup/domain.com/log.week
```



