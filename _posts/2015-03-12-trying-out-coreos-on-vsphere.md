---
layout: post
comments: true
title: Trying out CoreOS on vSphere
---

[CoreOS](https://coreos.com/) is a minimal Linux distribution created to serve as a host for running [Docker](http://www.docker.com) containers. 

Since stable release 557, CoreOS comes packaged as an OVA and is supported as a guest OS on vSphere 5.5 and above. Deployment instructions can be found in [VMware KB 2109161](http://kb.vmware.com/kb/2109161).

vCenter customization of CoreOS is currently not supported and can be done only through `coreos-cloudinit`. This is a CoreOS implementation of [cloud-init](http://cloudinit.readthedocs.org/en/latest/), a mechanism for boot time customization of Linux instances, which can be used on vSphere with some manual effort. More info on `coreos-cloudinit` and its configuration files called `cloud-config` can be found in the [CoreOS Cloud-Config Documentation page](https://coreos.com/docs/cluster-management/setup/cloudinit-cloud-config/), while for a tutorial on how to make Cloud-Config work in vSphere take a look at [CoreOS with Cloud-Config on VMware ESXi](http://www.chrismoos.com/2014/05/28/coreos-with-cloud-config-on-vmware-esxi).

CoreOS comes with integrated `open-vm-tools`, an open source implementation of VMware Tools, which is quite handy since CoreOS doesn't offer a package manager or Perl, so no way to install VMware Tools after deployment. According to [VMware Compatibility Guide](https://www.vmware.com/resources/compatibility/search.php), the `open-vm-tools` package has recently become the recommended way for running VMware Tools on newer Linux distros and vSphere editions (e.g. RHEL/CentOS 7.x and Ubuntu >=14.04 on vSphere 5.5 and above). For more info on `open-vm-tools` head to [VMware KB 2073803](http://kb.vmware.com/kb/2073803).

As for CoreOS stable releases prior to 557, releases 522 and 494 are supported on vSphere as a [Technical Preview](http://kb.vmware.com/kb/2104303). Since they don't come prepackaged in a format that can be used directly on vSphere, their deployment involves one additional step - converting the downloaded .vmx file to OVF via [OVF Tool](https://developercenter.vmware.com/web/dp/tool/ovf). Check out [VMware KB 2104303](http://kb.vmware.com/kb/2104303) for detailed installation instructions.

As mentioned before, CoreOS doesn't offer a way to install additional packages directly to the OS, but provides __Toolbox__, which is by default a stock Fedora Docker container that can be used for installing sysadmin tools. Toolbox can be run using `/usr/bin/toolbox`, first execution of which will pull and run the container:

```
core@core557 ~ $ toolbox 
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