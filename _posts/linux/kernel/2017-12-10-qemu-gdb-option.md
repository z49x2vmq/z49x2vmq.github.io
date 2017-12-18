---
layout: post
title:  'Attaching Gdb to Running Qemu Guest for Kernel Debugging'
date:   2017-12-20 06:50 +0900
tags: linux gdb qemu kernel
---

Recently, while I was studying how socket(), listen(), and accept() functions work under the hood, I looked at the kernel code. There are so many struct types that keep track of states of so many things in the kernel code. From their names and the names of functions they are passed to I could guess what they. Using cscope("make cscope") and tags("make tags") I could navigate the sources. 

But still, I wanted to attach debugger like one can do to user land application. There were couple of ways to do so but using qemu with gdb stub seemed to suite my needs.

# Prerequisites
1. Working QEMU Guest VM
2. "vmlinux" iamge corresponds to the kernel used by the guest VM
3. (optional) Guest VM's kernel is built with heavy debug info, less optimization

# QEMU GDB Option
Enabling gdb stub can be easily done by appending `-s` option to "qemu" command. For instance:
```
bash$ qemu-system-x86_64 ... -s
```

`-s` option opens tcp port 1234 for gdb connection. From the host machine one can attach to the guest VM with gdb command, "target remote :1234".

```
(gdb) target remote :1234
Remote debugging using :1234

Thread 1 received signal SIGTRAP, Trace/breakpoint trap.
native_safe_halt () at ./arch/x86/include/asm/irqflags.h:54
```

Actually, `-s` option is a shorcut to more general option, `-gdb tcp::1234`. If you need to select other tcp port use `-gdb tcp::<port>` option instead.

Just attaching the gdb to the guest VM doesn't get you anywhere. You need to set a breakpoint at the code you want to examine and trigger the execution of that code in the guest VM.

# ddd
If you have knowledge about struct types used in the kernel gdb will be just fine. I don't have such a knowledge and I find data display feature of "ddd" very useful for studying various data types in the kernel.
