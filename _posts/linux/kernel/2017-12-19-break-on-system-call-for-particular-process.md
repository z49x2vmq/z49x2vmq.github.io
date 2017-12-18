---
layout: post
title:  'QEMU Guest OS Kernel 디버깅 시 System Call에 조건부 Breakpoint 걸기'
date:   2017-12-18 06:50 +0900
tags: linux gdb qemu kernel
---

Qemu Guest OS에서 프로그램을 실행하고 거기서 호출된 Syscall을 분석하고 싶을 경우. `SyS_<syscall>` 또는 `SYSC_<syscall>`에다가 breakpoint를 걸면 된다. 하지만 그냥 breakpoint만 걸어주면 엄청 많은 프로세스들이 breakpoint에 걸려서 불편하다.

조건부 Breakpoint로 Qemu Guest 시스템에서 돌아가는 득정 PID를 가진 Process에서 호출된 Syscall인 경우에만 Break가 걸리도록 해줘야한다. Linux Kernel은 각 CPU마다 특정 메모리 영역을 per_cpu 메모리라는 이름으로 할당을 한다. 그 영역에는 현재 CPU가 실행중인 Process의 정보를 담고있는 task_struct도 저장이 된다. 이 정보를 이용해서 현재 process의 PID를 조건으로 사용하는 breakpoint를 만들 수 있다.

먼저 Guest OS에서 돌릴 Process에 gdb를 물려서 Syscall이 호출되기 전 상황에서 break를 걸어 놓고, `info proc`으로 pid를 조회해야 된다.

```
# Guest OS gdb Session

(gdb) info proc
process 3159
...
```

그런 다음에 Host OS Qemu gdb 세션에서 조건부 breakpoint를 걸어주고 Continue를 해준다.

```
# Host gdb Session(attached to QEMU gdb server)

(gdb) break SyS_listen if (**(struct task_struct**)((uint64_t)__per_cpu_offset[$_thread-1] + (uint64_t)&current_task)).pid == 3159
Breakpoint 1 at 0xffffffff81693d10: SyS_listen. (2 locations)
(gdb) c
Continuing.
```

Guest OS gdb 세션에서 Syscall 전에 Break 걸려있는 프로세스를 Continue 해주면, Host gdb 세션에서 아래와 같이 Syscall에 break가 걸린다.

```
# Host gdb Session

...
Thread 1 hit Breakpoint 1, SyS_listen (fd=3, backlog=5) at net/socket.c:1477
(gdb)
```

여기서 부터 n,s,ni,si 등을 이용해서 커널이 어떻게 생겼는지 알아보면 된다.
