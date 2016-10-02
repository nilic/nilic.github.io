---
layout: post
comments: true
title: A few VMware Virtual Volumes (VVol) tips
---

VMware Virtual Volumes (VVols) are a vSphere feature which allow for storing VM virtual disks and other files natively on the storage system (instead of on VMFS datastores created on top of LUNs) and managing their placement through vSphere storage policies. VVols are available since vSphere 6.0 and require a vSphere Standard or Enterprise Plus license.

VVols are a pretty fascinating piece of technology, which I'm guessing are set to completely replace VMFS datastores in a few years time. Since they are a fairly new feature without many implementation tips available online, here are a few things to bear in mind when deploying VVols:

* in order to use VVols, besides a VVol certified storage system, HBAs in your ESXi hosts need to support Secondary LUN IDs - to check whether this is the case, refer to the the [VMware Compatibility Guide for IO Devices](http://www.vmware.com/resources/compatibility/search.php?deviceCategory=io) and be sure to select *Secondary LUNID (Enables VVols)* in the *Features* section
  * if your hosts are running ESXi 6, you can check for HBA VVol compatibility from the command line by running `esxcli storage core adapter list` and noting whether "Capabilites" column contains *Second Level Lun ID* for the vmhbas in question

* although maximum data VVol (= virtual disk) size in vSphere 6.0 is 62 TB, you should check what are the limits on the storage system side, because they could be much less and force you to create multiple VVol datastores on the same system in order to consume its full available capacity; one such example is EMC Unity which allows you to consume up to 16 TB of a single storage pool for a VVol datastore

* migration of VMs from VMFS to VVol datastores is performed using Storage vMotion and should be initiated **solely** from the web client; the vSphere C# client doesn't offer the option to select a storage policy in the Storage vMotion wizard, which results in user-created storage policies being ignored and migrated VMs always being assigned the default "VVol No Requirements Policy"

* if you are using Veeam Backup and Replication, VVols are supported since VBR 8.0U2b but with a few caveats:
  * direct SAN transport mode is not supported
  * Virtual Appliance (Hot Add) processing mode requires that all VBR proxy VM disks are located on the same VVol with the processed VM.
