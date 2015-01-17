---
layout: post
comments: true
title: Search available ESXCLI commands
---

A way to search through available ESXCLI commands from ESXi Shell, vCLI or vMA:

```
esxcli esxcli command list | grep <keyword>
```

E.g.

```
~ # esxcli esxcli command list | grep snmp
system.snmp                                             get         
system.snmp                                             hash        
system.snmp                                             set         
system.snmp                                             test
```
