---
layout: post
title: 'VMWare Workstation Player + Fedora 29'
date:   2019-02-14 09:00 +0900
tags: vmware open-vm-tools
---

# Auto Resize Problem
After clean installation of Fedora 29 on VMWare Player + installation of
open-vm-tools and open-vm-tools-desktop. Everything(Clipboard, file drag&drop,
etc) works fine. All except auto screen resize.

`xorg-x11-drv-vmware` package is needed for this to work.

```
dnf install xorg-x11-drv-vmware
```

# Share Directory Auto Mount
Unlike vmware-tools, serivce created by open-vm-tools does not mount the hgfs
when it starts.

Adding following entry to fstab will make shared directory automatically mount
during startup.

/etc/fstab:
```
.host:/   /mnt/hgfs/         fuse.vmhgfs-fuse uid=1000,gid=1000,allow_other,defaults 0 0
```
