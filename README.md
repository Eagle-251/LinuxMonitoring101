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

  - `ps aux | grep <program name>` is a useful way to get information about a running process
  - `ps aux | grep $USER` returns all the process owned be a specific user
  - `ps -f -U $USER` does the same as above but without the pipe or grep

- ### `top`

  top is a program that fufills a function similar to `ps` but provides an interactive, real time version.

  Examples:

  - `top -d 1s` Changes the default delay between updates from 3s to 1s
  - `top -o %MEM` sort displayed processes by ram usage
  - `top -u $USER` Show only processes belonging to the current user

- ### `htop`
  htop is an enhanced version of top using the [ncurses](https://en.wikipedia.org/wiki/Ncurses) TUI library to provide an interactive interface for the user including mouse support.

## Network

## Logs

Logging in most major Linux distributions is usually handled by journald, with some important exceptions for example apt/apt-get on Debian/Ubuntu does not log to systemd but it uses syslog.

The main difference being that journald logs to binary files that can be only read with the utility `journalctl` whereas syslog logs to [RFC5424](https://tools.ietf.org/html/rfc5424) compliant log files.

## User Management
