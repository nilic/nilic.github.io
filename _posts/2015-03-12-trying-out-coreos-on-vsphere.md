---
layout: post
comments: true
title: Trying out CoreOS on vSphere
---

[CoreOS](https://coreos.com/) is a minimal Linux distribution created to serve as a host for running [Docker](http://www.docker.com) containers. Here's some info on how to try it out on vSphere.

Since stable release 557, CoreOS comes packaged as an OVA and is supported as a guest OS on vSphere 5.5 and above. Deployment instructions can be found in [VMware KB 2109161](http://kb.vmware.com/kb/2109161) and they are pretty much straightforward, with the exception of the need to modify boot parameters in order to change the password for the `core` user before logging in for the first time.

vCenter customization of CoreOS is currently not supported and customization can be done only through `coreos-cloudinit`. This is a CoreOS implementation of [cloud-init](http://cloudinit.readthedocs.org/en/latest/), a mechanism for boot time customization of Linux instances, which can be employed on vSphere with some manual effort. More info on `coreos-cloudinit` and its configuration files called `cloud-config` can be found in the [CoreOS Cloud-Config Documentation page](https://coreos.com/docs/cluster-management/setup/cloudinit-cloud-config/), while for a tutorial on how to make Cloud-Config work in vSphere take a look at [CoreOS with Cloud-Config on VMware ESXi](http://www.chrismoos.com/2014/05/28/coreos-with-cloud-config-on-vmware-esxi).

CoreOS comes with integrated `open-vm-tools`, an open source implementation of VMware Tools, which is quite handy since CoreOS doesn't offer a package manager or Perl, so no way to manually install VMware Tools after deployment. According to [VMware Compatibility Guide](https://www.vmware.com/resources/compatibility/search.php), the `open-vm-tools` package has recently become the recommended way for running VMware Tools on newer Linux distros and vSphere editions (e.g. RHEL/CentOS 7.x and Ubuntu >=14.04 on vSphere 5.5 and above). For more info on `open-vm-tools` head to [VMware KB 2073803](http://kb.vmware.com/kb/2073803).

As for CoreOS stable releases prior to 557, releases 522 and 494 are supported on vSphere as a [Technical Preview](http://kb.vmware.com/kb/2015161). Since they don't come prepackaged in a format that can be used directly on vSphere, their deployment involves one additional step - converting the downloaded .vmx file to OVF via [OVF Tool](https://developercenter.vmware.com/web/dp/tool/ovf). Check out [VMware KB 2104303](http://kb.vmware.com/kb/2104303) for detailed installation instructions.

## CoreOS quickstart

After logging in for the first time, you'll probably want to start building and running containers. Obviously, Docker comes preinstalled and CoreOS currently uses BTRFS as the filesystem for storing images and containers:

```
$ docker -v
Docker version 1.4.1, build 5bc2ff8-dirty
$ docker info
Containers: 0
Images: 23
Storage Driver: btrfs
 Build Version: Btrfs v3.17.1
 Library Version: 101
Execution Driver: native-0.2
Kernel Version: 3.18.1
Operating System: CoreOS 557.2.0
CPUs: 2
Total Memory: 5.833 GiB
Name: core557
ID: 2BUV:642W:WTZQ:3L4O:FFIY:JOC5:XKO2:3QPC:ADEJ:LSCS:QS5K:XHKB
```

As mentioned before, CoreOS doesn't offer a way to install additional packages directly to the OS, but provides __Toolbox__, which is by default a stock Fedora Docker container that can be used for installing sysadmin tools. Toolbox can be run using `/usr/bin/toolbox`, first execution of which will pull and run the container:

```
$ toolbox 
fedora:latest: The image you are pulling has been verified
00a0c78eeb6d: Pull complete 
834629358fe2: Pull complete 
834629358fe2: Pulling fs layer 
Status: Downloaded newer image for fedora:latest
core-fedora-latest
Spawning container core-fedora-latest on /var/lib/toolbox/core-fedora-latest.
Press ^] three times within 1s to kill container.
-bash-4.3#
```

After that, packages are `yum install <package>` away, while CoreOS filesystem is mounted inside the container to `/media/root`.

## Further reading

Compared to traditional Linux distributions, CoreOS has a fundamentally different approach to perfoming updates, which involves automatic upgrades of the complete OS as soon as the new release is available . Therefore, take a look at the [CoreOS Update Philosophy](https://coreos.com/using-coreos/updates/) in order not to be surprised when your container hosts start automatically upgrading and rebooting themselves.

For running containers at scale, check out CoreOS documentation pages on [etcd](https://coreos.com/using-coreos/etcd/) and [fleet](https://coreos.com/using-coreos/clustering/).