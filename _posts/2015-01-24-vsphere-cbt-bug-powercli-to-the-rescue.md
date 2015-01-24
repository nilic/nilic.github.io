---
layout: post
comments: true
title: vSphere CBT bug (and PowerCLI to the rescue)
---

A few months ago, VMware published information about a pretty catastrophic vSphere bug described in [VMware KB 2090639](http://kb.vmware.com/kb/2090639). TLDR version: if you have expanded a CBT-enabled .vmdk past any 128GB boundary (e.g. 128GB, 256GB, 512GB etc.), all your backups since are possibly corrupted.

The above KB article provides workaround information for this issue, which involves resetting CBT for the affected VMs. This can be accomplished either via shutting down the VM and editing its Configuration Parameters or on-the-fly via PowerCLI / VDDK API. [Veeam KB 1940](http://www.veeam.com/kb1940) provides PowerCLI code which can be employed for resetting CBT for all CBT-enabled VMs in the vCenter inventory, without the need for shutting them down first.

I have adapted this code to a quick-and-dirty PowerCLI script, which can be found [here](https://github.com/nilic/powercli/blob/master/ResetCBT.ps1) (for those not familiar with Github, clicking on __raw__ will take you directly to the script).

Just take notice that the script will create and instantly delete a snapshot for _all_ CBT-enabled VMs, so it's probably a good idea not to execute the script during work hours. Also, the first following backup will last longer, since backup software won't be able to use CBT information and will have to scan the whole .vmdk for changed blocks.