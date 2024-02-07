---
layout: default
title: Helpful Linux Commands  
nav_order: 2 
# permalink: /topics/utils/target_execute
parent: Utilities
---

## Helpful Linux Commands

Status: Draft | Updated: {{ page.path | file_date | date_to_string }}



## [`watch`][WATCH]

*Periodically runs a chosen executable. <br/>
e.g: run `nvidia-smi` every two seconds with highlighting what has changed between consecutive runs*

```
watch -t -d -n 2 nvidia-smi
```


## [`rsync`][RSYNC]

*Synchronise and backup data*

```
rsync -vahPAX g@target.ip.v4.address:/target/path/ destination/directory/
rsync -vahPAX g@172.16.3.2:/home/g/Workspace g/

rsync -vahP /home/ganindu/Downloads/nvfw-dgxstationa100_23.3.1_230306.tar.gz    g@DGX-ganindu:/home/g/Downloads/


```

Options used in the example above:

```
-v : verbose
-a : archive (equals to -rlptgoD: recursive|replicate symlinks|preserve permissions|preserve modification times|preserve group|preserve owner|preserve device files and special files)
-h : human readable 
-P : progress
-A : preserve ACLs
-X : preserve extended attributes
```

<br />
 

## `nvidia-smi -q -u`

## `nvidia-smi topo -matrix` 

## `nvidia-smi topo -p2p rw`

## `nvidia-smi topo -p2p rw > dev_null && nvidia-smi nvlink -s`

## `sudo ipmitool sel list`

## `sudo ipmitool sensor`

## `sudo ipmitool sdr`

## `uptime -p` 

## `nvme list` 

# NVSM: Nvidia System Management CLI (for NVidia DGX)

NVSM is an "always-on" health monitoring engine which is meant to catch issues proactively. We an drop in to the nvsm utility with `nvsm`

within nvsm

* help (-h): for the documentation.

some sample frequently used commands 

* show health
* show version

## Update container 

You will need to go to the support portal and download the latest update container.

(Below is an example with a .run file)

```
# update all
./nvfw-dgxstationa100_22.2.1_220209.run update_fw all

# update GPU firmware
sudo ./nvfw-dgxstationa100_22.2.1_220209.run update_fw VBIOS

# show version information 
./nvfw-dgxstationa100_22.2.1_220209.run show_version

```


## Give group permissions to a directory 

1. create a group e,g. tms.bucher
    * `sudo groupadd tms.bucher`
    * `sudo usermod -aG tms.bucher user1`
    * `sudo usermod -aG tms.bucher user2`

2. then run `sudo chown -R  $(id -u):tms.bucher path/to/dir/` 

### giving read write execute permission to the directory to group members 

even though you changed ownerships the directiry may still not allow users to read, write or execute within that directory 

gives read write permissions 

```
sudo chmod -R g+rx path/to/dir/
```

gives execute permissions 
```
sudo chmod -R g+rwx path/to/dir/
```
### TLDR
in Unix and Linux file systems, permissions are represented both symbolically (e.g., rwxr-xr-x) and numerically (e.g., 755). The numeric representation is a three-digit octal number, with each digit representing a different set of users: the owner, the group, and others, in that order.

Each digit is the sum of its component bits in the following way:

Read (r) permission is 4.
Write (w) permission is 2.
Execute (x) permission is 1.
To form a digit for each user category, sum the permissions you want to grant. For example:

g+rx means you're adding read (4) and execute (1) permissions for the group, which sums up to 5. Since we're focusing on the group permissions, this corresponds to setting the group's permission digit to 5. If we consider default permissions for the owner as read, write, and execute (7) and no permissions for others (0), this would numerically be represented as 750.

g+rwx means you're adding read (4), write (2), and execute (1) permissions for the group, summing up to 7. Again, assuming full permissions for the owner (7) and none for others (0), this would be 770 numerically.

Here's a breakdown of how the octal representation maps to symbolic permissions:

0 = ---
1 = --x
2 = -w-
3 = -wx
4 = r--
5 = r-x
6 = rw-
7 = rwx
So, to apply these permissions using chmod with numerical values:

To give the group read and execute permissions, without changing the owner's permissions (assuming they're full) and ensuring others have no permissions, you would use:

```
sudo chmod 750 path/to/dir/
```
To give the group read, write, and execute permissions (full permissions), again without changing the owner's permissions and ensuring others have no permissions, you would use:

```
sudo chmod 770 path/to/dir/
```

These commands apply the permissions directly to the path/to/dir/ directory. If you want to apply them recursively to all contents within the directory as well, add the -R option to chmod, like chmod -R 750 path/to/dir/ or chmod -R 770 path/to/dir/.



## `sudo nvsm-health --show --log-level=debug`





## [tail] view dynamic logs 

Tail looks like a very basic tool with limited functinality but it nicely format logfiles and make it easier to view dynamic log files.

The command below follows a logfile wile showing the complete log file.
 


```
tail -n +1 -f logs/4e51108a-693e-423d-a7f9-94f2b8a5cd82.txt 
```

o see the last 100 lines of a log while following it, you can use the following tail command:
```
tail -n 100 -f log_file.txt
```
This command will start at the end of the log file and display the last 100 lines. As new lines are added to the log file, tail will continue to display them. To stop following the log file, press Ctrl+C.

Here is an example of how to use the tail command to follow the last 100 lines of a system log file:
```
tail -n 100 -f /var/log/syslog
```
This command will display the last 100 lines of the system log file, and will continue to display new lines as they are added to the log file.

You can also use the tail command to follow the last 100 lines of a log file that is being compressed in real time. To do this, you can use the following command:
```
tail -n 100 -f <(zcat log_file.txt.gz)
```
This command will decompress the log file on the fly and display the last 100 lines. As new lines are added to the log file and compressed, tail will continue to display them.

The tail command is a very powerful tool for monitoring log files in real time. It is especially useful for troubleshooting problems or tracking the progress of long-running tasks.


---
*Click [here][ERRORS-SUGGESTIONS] to report Errors, make Suggestions or Comments!*

[WATCH]: https://man7.org/linux/man-pages/man1/watch.1.html
[RSYNC]: https://man7.org/linux/man-pages/man1/rsync.1.html


[ERRORS-SUGGESTIONS]: https://github.com/ganindu7/deepnotes/issues

<!-- Latex in markdown -->
<script src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML" type="text/javascript"></script>
<!-- $$ \nabla_\boldsymbol{x} J(\boldsymbol{x}) $$ -->
