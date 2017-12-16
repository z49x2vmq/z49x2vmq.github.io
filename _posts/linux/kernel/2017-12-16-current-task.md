---
layout: post
title:  'Kernel 디버깅시 현재 Process(Task) Struct 조회 '
date:   2017-12-16 07:50 +0900
tags: linux kernel
---


아래 처럼 현재 커널에서 현재 처리중인 process를 확인 할 수 있다. Breakpoint 조건으로 .pid 값을 설정하면 qemue Guest OS에서 특정 Process의 System Call에서 Break가 걸리게 할 수 있다. 기타 등등 활용 방법은 다양하다.

```
(gdb) print **(struct task_struct**)((uint64_t)__per_cpu_offset[$_thread-1] + (uint64_t)&current_task)
```

* __per_cpu_offset: 각 CPU 마다 고유의 저장소를 가지고 있는데 __per_cpu_offset 배열이 CPU 별 고유 저장소 위치를 가지고 있다.
* current_task: __per_cpu_offset으로 부터 현재 process의 task_struct 구조체가 저장된 offset
