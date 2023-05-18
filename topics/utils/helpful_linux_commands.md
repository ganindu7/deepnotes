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
2. then run `sudo chown -R  $(id -u):tms.bucher path/to/dir/`











## `sudo nvsm-health --show --log-level=debug`








---
*Click [here][ERRORS-SUGGESTIONS] to report Errors, make Suggestions or Comments!*

[WATCH]: https://man7.org/linux/man-pages/man1/watch.1.html
[RSYNC]: https://man7.org/linux/man-pages/man1/rsync.1.html


[ERRORS-SUGGESTIONS]: https://github.com/ganindu7/deepnotes/issues

<!-- Latex in markdown -->
<script src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML" type="text/javascript"></script>
<!-- $$ \nabla_\boldsymbol{x} J(\boldsymbol{x}) $$ -->
