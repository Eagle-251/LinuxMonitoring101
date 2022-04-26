# WIP

# Monitoring Linux Systems

This is a collection of tools and snippets in relation to monitoring a running linux system. This is mainly focused on monitoring headless systems, but could still apply to systems running a desktop environment.

Areas of concern for monitoring a linux systems could be:

- [System Stats](#system-stats)
- [Process Monitoring](#process-monitoring)
- [Network](#network)
- [Logs](#logs)
- [User Management](#user-management)

## System stats

System statistics on a Linux system could refer to a number of things. It could be hardware statistics, processing resource use, energy use etc. There is a lot of overlap with [Process Monitoring](#process-monitoring) so in order to satisfy [DRY](https://wikiless.org/wiki/Don%27t_repeat_yourself?lang=en) I will focus on tools and methods to query static information about the Linux systems state.

<br>

- ### `uname`

  This command will _"Print certain system information"_ about the system.
  This command will provide generic static information about your system. FOr example the kernel version, the hostname, the instruction set architecture, processor type and the operating system type.
  This command can be very useful in scripts intended to be run on systems with differing architectures. For example, it could be used in a script to download the correct Goland binary for your system:

  - `https://go.dev/dl/go1.17.7.linux-$(uname -i).tar.gz"`

- ### `lshw`

  `lshw` is a cli tool to provide information on the current hardware configuration of your machine.
  For example, `sudo lshw -class network` will return information about configured NICs and network interfaces.
  The `-json` option can also be useful for accessing specific variables in scripts.
  <br>

- ### `lsusb`

  This command is similar to lshw but is returns information about USB buses on the system and any devices connected to them.

- ### `lspci`

  Very similar in output to `lsub`except this command is concerned with PCI devices

  Example:

  - `lspci -kvm` will give return all pci devices in a readable form with the drivers being used for each device included.

- ### `lsblk`

  This will print all block devices in a tree like format.

  Examples:

  - `lsblk -f` Print block filesystem information alongside block info
  - `lsblk --exclude 7` Exclude a specific device type, one integer defaults to the `MAJ` number aka device type
    This specific command will exclude snap loop devices from the command output.

- ### `df`

  This command gives information about filesystems in use by the system.

  Examples:

  - `df -h` returns filesystems with their usage in a human readable format
  - `df -h path/to/file` the same as above but will return the filesystem containing the path specified
  - `df -x tmpfs` exclude temporary filesystems from the output

- ### `du`

  This command is very useful as it will return disk usage for a specific file path and to a specific directory depth.

  Examples:

  - `du path/to/file` Recursively returns disk usage for all files from the specified path
  - `du -h --max-depth=2 path/to/file` Will output disk usage recursively a maximum of two levels down from the specified path and print the file sizes in human readable form
  - `du -sh path/to/file` Does the same as above except will only print the size of file/directories in the current directory

## Process Monitoring

- ### `ps`

  ps returns a list of running processes based on a scope set by the arguments provided to the program.

  - `ps <no args>` will return only the processes attached to the current session
  - `ps a` shows all processes belonging to the current user
  - `ps aux` shows all processes running on the machine

  Examples:

  - This is a useful way to get information about a running process: <br> `ps aux | grep <program name>`
  - Return all the process owned be a specific user: <br> `ps aux | grep $USER`
  - This does the same as above but without the pipe or grep: <br> `ps -f -U $USER`
  - Sort processes by memory use: <br> `ps aux --sort size`

- ### `top`

  top is a program that fufills a function similar to `ps` but provides an interactive, real time version.

  Examples:

  - `top -d 1s` Changes the default delay between updates from 3s to 1s
  - `top -o %MEM` sort displayed processes by ram usage
  - `top -u $USER` Show only processes belonging to the current user

- ### `htop`
  htop is an enhanced version of top using the [ncurses](https://en.wikipedia.org/wiki/Ncurses) TUI library to provide an interactive interface for the user including mouse support.

## Network

There are many linux utilities to monitor networks and network traffic.
They range from simple utilities that return the static state and/or configuration
of network interfaces to complex network monitoring tools.

- ### `ping`

  Ping is a simple but extremely useful tool for monitoring the network connectivity and latency of network of a linux system. It send `ICMP ECHO_REQUEST` packets to a specific host and waits for the reply. The time between the sending and receiving of the packet is returned to the user in ms. This value can be a useful proxy for the latency of a network.

  Examples:

  - Send pings to google.com every second until the user exits the program:<br>`ping google.com`

  - Send 4 pings to google.com then exit: <br> `ping -c 4 google.com`

  - Send 4 pings with an interval of 3 seconds between pings:<br> `ping -i 3 -c 4 google.com`

  - Ping google.com using a specific interface (useful to diagnosing the latency of vpn): <br> `ping -I <interface_name> google.com`

- ### `ip`

  `ip` is a utility to show and set routes, create and bring up/down network interfaces, devices and tunnels.
  The command has many "objects" that fulfill different functions:

  | Object                 | Use                                                 |
  | ---------------------- | --------------------------------------------------- |
  | address                | protocol (IP or IPv6) address on a device.          |
  | addrlabel              | label configuration for protocol address selection. |
  | l2tp                   | tunnel ethernet over IP (L2TPv3).                   |
  | link                   | network device.                                     |
  | maddress               | multicast address.                                  |
  | monitor                | watch for netlink messages.                         |
  | mptcp                  | manage MPTCP path manager.                          |
  | mroute                 | multicast routing cache entry.                      |
  | mrule                  | rule in multicast routing policy database.          |
  | neighbour              | manage ARP or NDISC cache entries.                  |
  | netns                  | manage network namespaces.                          |
  | ntable                 | manage the neighbor cache's operation.              |
  | route                  | routing table entry.                                |
  | rule                   | rule in routing policy database.                    |
  | tcp_metrics/tcpmetrics | manage TCP Metrics                                  |
  | token                  | manage tokenized interface identifiers.             |

  The most common of these however are address, route and link object. Most of the objects will return their configuration with no arguments and will allow you to `set` a config depending on the object called.

  Examples:

  - Return address information if all network interfaces on the system: <br> `ip address`

  - Show link layer information for all network interfaces: <br> `ip link`

  - Output the routing table for the system: <br> `ip route`

  - Add a new network interface: <br> `ip addr add ip/ dev <interface_name>`

  -

## Logs

Logging in most major Linux distributions is usually handled by journald, with some important exceptions for example apt/apt-get on Debian/Ubuntu does not log to systemd but it uses syslog.

The main difference being that journald logs to binary files that can be only read with the utility `journalctl` whereas syslog logs to [RFC5424](https://tools.ietf.org/html/rfc5424) compliant log files.

### Syslog

Syslog is both a logging daemon and logging messages format. Rsyslog is the daemon that handles log messages and is configured in `/etc/rsyslog.conf` and in drop-in configuration files in the `/etc/rsyslog.d` directory which are added via an "include" in the main conf file.
Here specific locations for logs from specific daemons or applications are set.

For example:

`auth,authpriv.* /var/log/auth.log`

This line in `/etc/rsyslog.conf` tell syslog to store logs from authentication utilities (eg pam, sudo) to the `auth.log`.

### Journald

Journald is logging component of the [systemd](https://wikiless.org/wiki/Systemd?lang=en) init system. Systemd is utilises plain text `unit` files of various types to manage the state of the system at various stages of the boot process. These stages are known as `targets`. The system moves progressively through each of these targets until it reaches the last target, usually the `user-session` target.

Each systemd unit has it's own log which can be accessed through the `journalctl` binary.
Unlike syslog, journalctl (when executed with no arguments) can return logs of all the systemd units in one stream. <br>
You can specify a particular unit file however by using the `-u` flag:

- `journalctl -u avahi-daemon.service`

Including the `-f` will follow log output in real time:

- `journalctl -fu avahi-daemon.service`

Journald also splits up logs by boots. This will show all logs from the current boot:

- `journalctl -b`

## User Management
