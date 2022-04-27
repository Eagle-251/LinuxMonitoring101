## Initial Checks

### SSH

#### Preparation

For remote ssh I use my an [authentication subkey](https://www.linode.com/docs/guides/gpg-key-for-ssh-authentication/) of my [pgp key](https://gilchrist.scot/.well-known/openpgpkey/hu/esfzao74zw7on5f8ickbk4htbj1js7nb) in place of a regular ssh key.

As I use [fish](fishshell.com/), I load the `gpg-agent` in place of `ssh-agent`, set the correct `tty` and launch `gpg-agent`

```fish
set -e SSH_AUTH_SOCK
set -U -x SSH_AUTH_SOCK (gpgconf --list-dirs agent-ssh-socket)

set -x GPG_TTY (tty)

gpgconf --launch gpg-agent
```

On the server, I put the output of `gpg --export-ssh-key ewan@gilchrist.scot` into `~/.ssh/authorized_keys`. This will me to login to the server using my gpg key.

I also have an entry in `.ssh/config` for this server:

```
Host gilchrist.scot
	Hostname <vpn-ip>
	User ewan
	Port <non-standard-port>

```

#### Test connection

In order to do any monitoring I need to be logged in. This also is an important check for the [vpn](https://tailscale.com/) that I use to connect.

`ssh gilchrist.scot`

Output:

```shell
Linux gilchrist.scot 4.19.0-18-amd64 #1 SMP Debian 4.19.208-1 (2021-09-29) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Wed Apr 27 09:24:20 2022 from **.**.**.**

Welcome to fish, the friendly interactive shell
ewan@gilchrist.scot ~>
```

#### Perform basic system checks

Using tools such as [`htop`](/README.md#htop), [`systemctl`](/README.md#journald) and others I check the running of the server.

_htop_:

```
  Avg[###************                                                                                        13.7%]   5  [                                                                                                        0.0%]
  1  [######################********************************************                                     60.0%]   6  [                                                                                                        0.0%]
  2  [############************************************                                                       44.4%]   7  [                                                                                                        0.0%]
  3  [                                                                                                        0.0%]   8  [                                                                                                        0.0%]
  4  [                                                                                                        0.0%]   Tasks: 637, 2963 thr; 1 running
  Mem[|||||||||||||||||||||||||||||||||||||||||||||||||||||#*************                              15.4G/31.4G]   Load average: 2.41 2.17 2.13
  Swp[                                                                                                       0K/0K]   Uptime: 96 days, 15:41:33
  PID USER      PRI  NI  VIRT   RES   SHR S CPU% MEM%   TIME+  Command
  561 root       20   0 2166M 1781M  2628 S  0.0  5.5  0:00.00 /usr/lib/udisks2/udisksd
  568 root       20   0 2166M 1781M  2628 S  0.0  5.5  0:08.49 /usr/lib/udisks2/udisksd
  578 root       20   0 2166M 1781M  2628 S  0.0  5.5  0:00.00 /usr/lib/udisks2/udisksd
  588 root       20   0 2166M 1781M  2628 S  0.0  5.5  3:05.62 /usr/lib/udisks2/udisksd
  545 root       20   0 2166M 1781M  2628 S  0.0  5.5  9h10:15 /usr/lib/udisks2/udisksd
24429 _apt       30  10 1307M 1256M  1192 S  0.0  3.9  0:00.08 clamd
15252 _apt       30  10 1307M 1256M  1192 S  0.0  3.9  3:03.59 clamd
23062 matrix     20   0 4279M  921M  916M S  0.0  2.9  1:21.87 postgres: synapse synapse 172.18.0.12(51604) idle
23086 matrix     20   0 4278M  914M  908M S  0.0  2.8  1:18.56 postgres: synapse synapse 172.18.0.12(51746) idle
23087 matrix     20   0 4278M  912M  906M S  0.0  2.8  1:12.95 postgres: synapse synapse 172.18.0.12(51748) idle
23093 matrix     20   0 4279M  903M  897M S  0.0  2.8  1:14.41 postgres: synapse synapse 172.18.0.12(51768) idle
23063 matrix     20   0 4278M  903M  897M S  0.0  2.8  1:25.50 postgres: synapse synapse 172.18.0.12(51606) idle
23089 matrix     20   0 4278M  902M  896M S  0.0  2.8  1:14.23 postgres: synapse synapse 172.18.0.12(51752) idle
23196 matrix     20   0 4278M  897M  891M S  0.0  2.8  1:12.76 postgres: synapse synapse 172.18.0.12(52148) idle
23091 matrix     20   0 4278M  896M  890M S  0.0  2.8  1:16.67 postgres: synapse synapse 172.18.0.12(51754) idle
23204 matrix     20   0 4278M  894M  888M S  0.0  2.8  1:18.16 postgres: synapse synapse 172.18.0.12(52152) idle
19540 matrix     20   0 4272M  653M  651M S  0.0  2.0  0:42.10 postgres: checkpointer
23056 matrix     20   0 4278M  617M  612M S  0.0  1.9  1:13.75 postgres: synapse synapse 172.18.0.12(51602) idle
23054 matrix     20   0  910M  488M 11456 S  0.0  1.5  1:24.53 /usr/local/bin/python -m synapse.app.homeserver -c /data/homeserver.yaml
23055 matrix     20   0  910M  488M 11456 S  0.0  1.5  1:38.71 /usr/local/bin/python -m synapse.app.homeserver -c /data/homeserver.yaml
23057 matrix     20   0  910M  488M 11456 S  0.0  1.5  1:20.16 /usr/local/bin/python -m synapse.app.homeserver -c /data/homeserver.yaml
23058 matrix     20   0  910M  488M 11456 S  0.0  1.5  1:30.07 /usr/local/bin/python -m synapse.app.homeserver -c /data/homeserver.yaml
23059 matrix     20   0  910M  488M 11456 S  0.0  1.5  1:29.18 /usr/local/bin/python -m synapse.app.homeserver -c /data/homeserver.yaml
23060 matrix     20   0  910M  488M 11456 S  0.0  1.5  0:00.17 /usr/local/bin/python -m synapse.app.homeserver -c /data/homeserver.yaml
23061 matrix     20   0  910M  488M 11456 S  0.0  1.5  0:00.48 /usr/local/bin/python -m synapse.app.homeserver -c /data/homeserver.yaml
23064 matrix     20   0  910M  488M 11456 S  0.0  1.5  0:00.29 /usr/local/bin/python -m synapse.app.homeserver -c /data/homeserver.yaml
23065 matrix     20   0  910M  488M 11456 S  0.0  1.5  0:00.45 /usr/local/bin/python -m synapse.app.homeserver -c /data/homeserver.yaml
```

`systemctl status`:

```
● gilchrist.scot
    State: running
     Jobs: 0 queued
   Failed: 0 units
    Since: Thu 2022-01-20 17:24:54 CET; 3 months 5 days ago
   CGroup: /
           ├─user.slice
           │ └─user-1000.slice
           │   ├─user@1000.service
           │   │ ├─init.scope
           │   │ │ ├─11397 /lib/systemd/systemd --user
           │   │ │ └─11398 (sd-pam)
           │   │ └─dbus.service
           │   │   ├─25623 /usr/bin/dbus-daemon --session --address=systemd: --nofork --nopidfile --systemd-activation --syslog-only
           │   │   └─25626 /usr/bin/gnome-keyring-daemon --start --foreground --components=secrets
           │   ├─session-26777.scope
           │   │ ├─11434 tmux
           │   │ ├─11882 -fish
           │   │ └─12578 -fish
           │   └─session-29353.scope
           │     ├─  320 sshd: ewan@pts/0
           │     ├─  322 -fish
           │     ├─16767 sudo systemctl status
           │     ├─16768 systemctl status
           │     ├─16770 pager
           │     └─32694 sshd: ewan [priv]
           ├─init.scope
           │ └─1 /lib/systemd/systemd --system --deserialize 34
           └─system.slice
             ├─fail2ban.service
             │ └─11207 /usr/bin/python3 /usr/bin/fail2ban-server -xf start
             ├─matrix-postgres.service
            etc...
```
