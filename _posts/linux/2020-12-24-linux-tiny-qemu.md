---
layout: post
title:  "Linux tinyconfig and Qemu"
#date:   2020-12-24 17:50:01 +0900
---

Building Linux with tinyconfig looked tempting at first. But it has many
drawbacks due to lack of features. 

Many program that require basic feature from kernel doesn't work. Keep adding
more options make it bigger and bigger.

# Build kernel with tinyconfig.

```
# make tinyconfig
# make
```

# Qemu with the kernel
```
qemu-system-x86_64 \
    -kernel arch/x86_64/boot/bzImage \
    -append 'root=/dev/vda rw nokaslr init=/bin/bash' \
    -drive file=fedora.raw,if=virtio,index=0,format=raw,cache=directsync \
    -enable-kvm \
    -m 2g \
    -nographic \
    -s
```
Nothing comes up on screen.

# minimal config to boot. (ext4 rootfs + virtio)
## 64-bit support
```
[*] 64-bit kernel
```

## gdb
"next" from breakpoint somehow ends up in some kind of linux interrupt handler.
Below options prevent that.

```
> General setup
[ ] Embedded system
[ ] Configure standard kernel features (expert users)  ----
> Processor type and features
[*] Support x2apic
[*] Linux guest support  --->
> Processor type and features > Linux guest support
[*]   Enable paravirtualization code
[*]   KVM Guest support (including kvmclock) (NEW)
```

## Console output
Running the kernel in Qemu prints out nothing in the screen.
```
> Device Drivers > Character devices > Serial drivers
[*] 8250/16550 and compatible serial support
[*]   Console on 8250/16550 and compatible serial port
```

## Storage
virtio storage device and file system support
```
> Device Drivers
[*] PCI support  --->
> Device Drivers > Virtio drivers
[*]   PCI driver for virtio devices
[*]     Support for legacy virtio draft 0.9.X and older devices (NEW)
> Device Drivers > Block devices
[*]   Virtio block driver
> File systems
[*] The Extended 4 (ext4) filesystem
[*]   Use ext4 for ext2 file systems (NEW)
[*]   Ext4 POSIX Access Control Lists
```

## Make kernel know how to execute elf binary
What kind of binary format does it support without this?
```
> Executable file formats
[*] Kernel support for ELF binaries
[*] Write ELF core dumps with partial segments (NEW)
```

# Boot
It boots up and bash is executed.
```
Run /bin/bash as init process
random: fast init done
bash: cannot set terminal process group (-1): Inappropriate ioctl for device
bash: no job control in this shell
bash-5.0#
```

