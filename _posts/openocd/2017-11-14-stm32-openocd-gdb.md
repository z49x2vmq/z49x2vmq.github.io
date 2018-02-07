---
layout: post
title:  "OpenOCD와 gdb를 이용한 stm32 디버깅"
date:   2017-11-14 22:33:01 +0900
category: openocd
tags: [gdb, openocd, stm32, l432kc]
---

---
# 관련 포스트
{% for post in site.categories.openocd %} 
+ [{{ post.title }}]({{ post.url }}) {% endfor %}

---

STM 보드는 기본적으로 st-link가 달려있어서 디버깅 하기가 참 좋다.

# STM32 NUCLEO-L432KC 보드 OpenOCD 연결
```
$ ls -l openocd/scripts/board/st_nucleo_l*
-rw-r--r-- 1 root root 268 Jul 28 23:21 openocd/scripts/board/st_nucleo_l1.cfg
-rw-r--r-- 1 root root 302 Jul 28 23:21 openocd/scripts/board/st_nucleo_l476rg.cfg
```
기본 script 중에 l432kc 보드를 직접 지원하는 파일은 없지만 그냥 대충 ```st_nucleo_l476rg.cfg```를 사용하면 될 것 같다.

```
$ openocd -f board/st_nucleo_l476rg.cfg
Open On-Chip Debugger 0.10.0 (2017-07-28-23:20)
Licensed under GNU GPL v2
For bug reports, read
        http://openocd.org/doc/doxygen/bugs.html
Info : The selected transport took over low-level target control. The results might differ compared to plain JTAG/SWD
adapter speed: 500 kHz
adapter_nsrst_delay: 100
none separate
srst_only separate srst_nogate srst_open_drain connect_deassert_srst
Info : Unable to match requested speed 500 kHz, using 480 kHz
Info : Unable to match requested speed 500 kHz, using 480 kHz
Info : clock speed 480 kHz
Info : STLINK v2 JTAG v25 API v2 SWIM v14 VID 0x0483 PID 0x374B
Info : using stlink api v2
Info : Target voltage: 3.249670
Info : stm32l4x.cpu: hardware has 6 breakpoints, 4 watchpoints
```
된다!!!

일단 openOCD가 실행되면 tcp port 3개가 열린다. Tcp 3333은 gdb로 접속하는 포트, 4444는 telnet으로 접속해서 이것저것 할 수 있고, 6666은 뭔지 잘 모르겠다.
```
$ netstat -anp | grep openocd
(Not all processes could be identified, non-owned process info
 will not be shown, you would have to be root to see it all.)
tcp        0      0 0.0.0.0:4444            0.0.0.0:*               LISTEN      9741/openocd        
tcp        0      0 0.0.0.0:3333            0.0.0.0:*               LISTEN      9741/openocd        
tcp        0      0 0.0.0.0:6666            0.0.0.0:*               LISTEN      9741/openocd
```

먼저 telnet으로 붙어보면
```
$ telnet localhost 4444
Trying 127.0.0.1...
Connected to localhost.
Escape character is '^]'.
Open On-Chip Debugger
> help
adapter_khz [khz]
      With an argument, change to the specified maximum jtag speed.  For
      JTAG, 0 KHz signifies adaptive  clocking. With or without argument,
      display current setting. (command valid any time)
adapter_name
      Returns the name of the currently selected adapter (driver) (command
      valid any time)
...
```
`help`를 치면 뭔가 엄청 많은 걸 할 수 있다. `help`를 했을때 보여지는 command들은 gdb에서도 `monitor <command> ...`으로 사용이 가능하다고 한다.

이제 진짜로 gdb로 접속을 해보자
```
$ arm-none-eabi-gdb 
(gdb) file build/thp_makefile.elf 
Reading symbols from /build/thp_makefile.elf...done.
(gdb) target remote :3333
Remote debugging using :3333
0x08003cc2 in HAL_Delay (Delay=Delay@entry=60000) at /STM32Cube/Repository/STM32Cube_FW_L4_V1.10.0/Drivers/STM32L4xx_HAL_Driver/Src/stm32l4xx_hal.c:340
340       while((HAL_GetTick() - tickstart) < wait)
(gdb) load
Loading section .isr_vector, size 0x190 lma 0x8000000
Loading section .text, size 0x4850 lma 0x80001c0
Loading section .rodata, size 0x1f8 lma 0x8004a10
Loading section .ARM, size 0x8 lma 0x8004c08
Loading section .init_array, size 0x8 lma 0x8004c10
Loading section .fini_array, size 0x8 lma 0x8004c18
Loading section .data, size 0x68 lma 0x8004c20
Start address 0x80049a0, load size 19544
Transfer rate: 24 KB/sec, 2443 bytes/write.
(gdb) break main
Breakpoint 1 at 0x8002790: file Src/main.c, line 97.
(gdb) c
Continuing.
Note: automatically using hardware breakpoints for read-only addresses.

Breakpoint 1, main () at Src/main.c:97
97        HAL_Init();
(gdb)
```
먼저 gdb로 보드에 프로그램을 구울수도 있는데 `file <image>` 커맨드를 이용해서 debugging에 사용할 이미지를 고를 수 있다. `target remote :3333`은 아까 openocd를 실행하면 열리는 localhost에 3333 port로 디버깅 세션을 연결하라는 커맨드이다. 만약 Nucleo보드가 다른 컴퓨터에 연결되어있고 그 쪽에서 openocd가 실행 되있다면 `target remote [<hostname>|<ip>]:3333`으로 다른 컴퓨터에 디버깅 세션을 연결할 수도있다.

다음으로 `load` 커맨드는 디버깅에 사용될 이미지 파일을 디버깅 대상 보드에 구우라는 커맨드이다. 굽고나면 보드가 reset 된 상태로 break 상태가 되는 것 같다. `break main`은 main 함수 시작하는 코드에 breakpoint를 설정해서 프로그램이 실행이 해당 위치에 도달 하면 실행을 중단하고 이것저것 debugging을 할 수 있는 상태가 된다.

```
192         cksum += temp_humid[2] + temp_humid[0];
193
194         if (rc != 0) {
195           continue;
196         }
197
198         ret = PMS7003_Read(&huart2, &readings, OP_MODE_PASSIVE);
199         if (ret == PMS7003_OK) {
200
201           num += sprintf(&buf[num], "id:%s,", dId);
(gdb) advance 199
main () at Src/main.c:199
199         if (ret == PMS7003_OK) {
(gdb) print ret
$9 = PMS7003_OK
(gdb) print readings
$10 = {pm1_0_cf = 2, pm2_5_cf = 4, pm10_0_cf = 4, pm1_0_atmo = 2, pm2_5_atmo = 4, pm10_0_atmo = 4, pm_0_3_raw = 405, pm_0_5_raw = 125, pm_1_0_raw = 24, pm_2_5_raw = 2, 
  pm_5_0_raw = 0, pm_10_0_raw = 0}
(gdb) c
Continuing.
```
`advance 199`는 현재 source file의 199번 줄 바로 전까지 실행하다가 거기서 멈추라고 하는 것. `print ret`을 통해서 `PMS7003_Read()`함수의 return 값을 쉽게 확인 할 수도 있고. struct type인 readings의 각 필드값도 아주 쉽게 확인이 가능하다. 마지막 `c`는 `continue`의 약자로 다른  breakpoint에 도달하기까지 프로그램을 쭉 수행하도록 해준다.

gdb에 강력한 기능이 더 많이 있지만 이정도 만으로도 도대체 어디서부터 꼬인건지 찾아 내는 과정을 엄청 쉽게 만들어준다.


