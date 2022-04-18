# watch-auth-log

`watch-auth-log` is one of those tiny scripts to keep my private server happy and purring. The purpose is to:

- Watch `/var/log/auth.log`,
- Find lines that indicate a suspect SSH login attempt,
- Extract the offending IP address,
- Set a temporary `iptables` block on that address.

`watch-auth-log` internally calls the two utilities:
- `tail -F` to ensure that the log file monitoring continues even when the file is "rotated"
- `iptables` to manipulate the firewall.

## Installation

Just copy the script to an appropriate location. I use `/root/bin` but anything will do.

## Activation & customizing

Try `watch-auth-log -h` to see an overview of the flags.

- `-c CHAIN` to specify the iptables chain to operate on; defaults to `INPUT`,
- `-l LOGFILE` to specify the logfile to watch; defaults to `/var/log/auth.log`,
- `-p port` to specify what port to block when locking out an IP address; defaults to 22 (the `ssh` port),
- `-t` for try-out mode; use this to check what `watch-auth-log` would do without actually setting or clearing blocks,
- `-T TTLSEC` to specify the duration of blocks; defaults to 900 (15 minutes),
- `-w WHITELIST` to specify which IPs never to block, separate IPs are given as one comma-separated string (ex.: `112.13.44.215,313.23.223.36`).

The lines that indicate a suspect SSH login attempt are assumed to be in the format:

```
Apr 18 10:47:37 server sshd[21208]: Failed password for invalid user guest from 43.134.206.118 port 55884 ssh2
```

The relevant match tag is `Failed password for invalid user ... from`. The offending IP is expected after that. If your server logs the ssh activity in a different format, then you'll need to adapt `watch-auth-log` for that format.

`watch-auth-log` must be run as user `root`, else it lacks the permission to scan logs and modify iptables. Downloading some odd script and running it as `root` on your system is... ummm at best not recommended. So you should obviously carefully review what this script does and check that it isn't malicious.

## Automatic startup

If you want this to utility to automatically start up after a reboot, then add the following to your root's crontab:

```
PATH=/bin:/usr/bin:/usr/local/bin:/sbin:/usr/sbin:/usr/local/sbin
@reboot sleep 60 && /root/bin/watch-auth-log 2>&1 | logger
```

Notes:

- Change `/root/bin/` to the actual directory where you place `watch-auth-log`.
- Piping the output to `logger` makes the actions visible via system logging, usually in `/var/log/messages`. Instead you can of course redirect these messages to a file.
- The `PATH` statement is needed to ensure that `watch-auth-log` finds the programs it uses internally, `tail` and `iptables`.
