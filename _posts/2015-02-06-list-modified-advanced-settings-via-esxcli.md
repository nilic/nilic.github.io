---
layout: post
comments: true
title: List host's modified advanced settings via ESXCLI
---

Just wanted to share a great tip by [Ather Beg](http://atherbeg.com/) on how to display all of ESXi host's advanced settings which are different than the default values via ESXCLI:

```
esxcli system settings advanced list -d
```

This will return a list of advanced settings similar to:

```
...

   Path: /Net/TcpipHeapSize
   Type: integer
   Int Value: 32
   Default Int Value: 0
   Min Value: 0
   Max Value: 32
   String Value: 
   Default String Value: 
   Valid Characters: 
   Description: Initial size of the tcpip module heap in megabytes. (REQUIRES REBOOT!)

   Path: /NFS/MaxVolumes
   Type: integer
   Int Value: 256
   Default Int Value: 8
   Min Value: 8
   Max Value: 256
   String Value: 
   Default String Value: 
   Valid Characters: 
   Description: Maximum number of mounted NFS volumes. TCP/IP heap must be increased accordingly (Requires reboot)

...
```

where __Int Value__ / __String Value__ (depending whether the parameter stores its value as integer or string) shows the current value for the parameter, while __Default Int Value__ / __Default String Value__ show its default value.

Source: [ProTip: How to remind yourself of Advanced Settings changes in ESXi](http://atherbeg.com/2014/11/10/protip-how-to-remind-yourself-of-advanced-settings-changes-in-esxi/)