---
layout: post
comments: true
title: Downloading files with wget on ESXi
---

A limited set of familiar POSIX-like tools and utilities is available on ESXi, courtesy of the [busybox](http://www.busybox.net/) executable:

```
~ # /usr/lib/vmware/busybox/bin/busybox --list
[
[[
addgroup
adduser
ash
awk
basename
cat
chgrp
chmod
chown
chvt
cksum
clear
cp
crond
cut
date
dd
delgroup
deluser
diff
dirname
dnsdomainname
du
echo
egrep
eject
env
expr
false
fdisk
fgrep
find
getty
grep
groups
gunzip
gzip
halt
head
hexdump
hostname
inetd
init
kill
ln
logger
login
ls
lzop
lzopcat
md5sum
mkdir
mkfifo
mknod
mktemp
more
mv
nohup
nslookup
od
passwd
poweroff
printf
readlink
reboot
reset
resize
rm
rmdir
sed
seq
setsid
sh
sha1sum
sha256sum
sha512sum
sleep
sort
stat
stty
sum
sync
tail
tar
tee
test
time
timeout
touch
true
uname
uniq
unlzop
unzip
usleep
vi
watch
wc
wget
which
who
xargs
zcat
```

As you can see, one of the tools present is `wget` which can be used for downloading files (e.g. installation ISOs, VIBs, offline bundles..) directly from the ESXi Shell, instead of first downloading locally to your desktop or jumphost and then uploading to hosts or datastores.

First, connect to ESXi Shell over SSH or DCUI and `cd` into the destination directory, which can be e.g. a shared datastore available to all hosts in your cluster:

```
cd /vmfs/volumes/datastore_name_here
```

or host's `/tmp` directory (common choice for VIBs and offline bundles since it gets emptied on reboot). After that, just fire wget away as `wget <file URL>`:

```
~ # cd /vmfs/volumes/ISO_images/
.. # wget http://releases.ubuntu.com/14.04.1/ubuntu-14.04.1-server-amd64.iso
Connecting to releases.ubuntu.com (91.189.92.163:80)
ubuntu-14.04.1-serve 100% |*****************************|   572M  0:00:00 ETA
```

Here we downloaded Ubuntu Server installation ISO directly to our "ISO_images" datastore.

## Direct installation of VIBs from an URL

Downloading installation ISOs is far from a best practice, since it's probably not the best idea to use your host's resources for downloading large files from the Internet, but the wget approach can save you same time if you're often manually installing VIBs or offline bundles on your ESXi hosts. 

For VIB files, an alternative to wget-ing and then installing the VIB would be to directly supply the URL of the VIB file to the `esxcli software vib install` command:

```
esxcli software vib install -v <URL of the VIB file>
```

e.g.

```
/tmp # esxcli software vib install -v http://vibsdepot.v-front.de/depot/DAST/iperf-2.0.5/iperf-2.0.5-1.x86_64.vib --no-sig-check
Installation Result
   Message: Operation finished successfully.
   Reboot Required: false
   VIBs Installed: DAST_bootbank_iperf_2.0.5-1
   VIBs Removed: 
   VIBs Skipped: 
```

Here we installed [hypervisor.fr](http://www.hypervisor.fr/) iperf for ESXi VIB directly from the [V-Front Software Depot](http://vibsdepot.v-front.de/wiki/index.php/Welcome). `--no-sig-check` is there to bypass signature verification, since the iperf VIB we are installing doesn't have one.

As `esxcli software vib install --help` informs us, the supplied URL can be HTTP, HTTPS or FTP. URLs can be supplied only to the command's `-v` switch, so this approach is limited to VIB files and not available for offline bundles.