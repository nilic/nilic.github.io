---
layout: post
comments: true
title: NetApp cDOT and ESXi SCSI UNMAP support
---

I have recently noticed that LUNs created on NetApp cDOT storage systems by default don't support space reclamation (aka SCSI UNMAP) which is a part of the vSphere VAAI Thin Provisioning primitive:

```
~ # esxcli storage core device vaai status get
naa.600a0980424733322f5d444f76513730
   VAAI Plugin Name: VMW_VAAIP_NETAPP
   ATS Status: supported
   Clone Status: supported
   Zero Status: supported
   Delete Status: unsupported
```

and when you try to reclaim space, you are greeted with the following message:

```
~ # esxcli storage vmfs unmap -l cdot_01
Devices backing volume 549a787c-f5a4021c-e59d-68b599cbd47c do not support UNMAP
```

Based on a short review of a few systems, this seems to be the case only for the cDOT platform (tested on Ontap 8.2 and 8.3), while LUNs created on 7-mode systems show `Delete Status: supported`.

Explanation for this behavior can be found in the NetApp cDOT official documentation, namely the [SAN Administration Guide](https://library.netapp.com/ecmdocs/ECMP1196784/html/GUID-93D78975-6911-4EF5-BA4E-80E64B922D09.html). The document states that in order for space reclamation to be supported on a LUN:

1. LUN needs to be thinly provisioned on the storage system
2. `space-allocation` option needs to be enabled on the LUN

Information on how to enable `space-allocation` on a LUN is available in the [same guide](https://library.netapp.com/ecmdocs/ECMP1196784/html/GUID-6AD84908-041A-497D-95A7-BB6AFDD1B282.html):

```
lun modify -vserver <vs> -volume <vol> -lun <lun> -space-allocation enabled
```

What's a bugger is that the LUN needs to be **offline** in order to run this command, so best thing to do would be to enable this prior to presenting the LUN to the hosts.

One more thing to have in mind is that the [VMware Compatibility Guide](http://www.vmware.com/resources/compatibility/search.php) explicitly states that "VAAI Thin Provisioning Space Reclamation is not supported" for ESXi >= 5.5, and Data Ontap < 8.3 (both clustered and 7-mode). So, even if your NetApp LUNs are configured properly for space reclamation, be sure that you are running a supported configuration before initiating a SCSI UNMAP operation from an ESXi host.