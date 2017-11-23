---
layout: post
title: 'aapcs : Parameter Passing'
date:   2017-11-22 06:47:01 +0900
tags: arm gcc gdb aapcs
categories: AAPCS
---
기본적으로 r0-r3 4개의 register 함수 호출시 파라미터로 사용을 한다. 함수 파라미터가 4개 이상인 경우는 stack에 push를 해주게 된다.

```c
#include<stdint.h>

void param_four (uint8_t one, uint16_t two, uint32_t three, uint32_t four);
void param_eight(uint8_t one, uint16_t two, uint32_t three, uint32_t four,
                 uint8_t five, uint16_t six, uint32_t seven, uint32_t eight);

void caller() {
  param_four (0xaa, 0xbbaa, 0xccbbaa, 0xddccbbaa);
  param_eight(0xaa, 0xbbaa, 0xccbbaa, 0xddccbbaa,
              0x11, 0x2211, 0x332211, 0x44332211);
}

void param_four(uint8_t one, uint16_t two, uint32_t three, uint32_t four) {
  return;
}

void param_eight(uint8_t one, uint16_t two, uint32_t three, uint32_t four,
                 uint8_t five, uint16_t six, uint32_t seven, uint32_t eight) {
  return;
}
```
4개의 파라미터 값을 받는 함수, `param_four()`,와 8개의 파라미터 값을 받는 함수, `para_eight()`. 파라미터는 8, 16, 32 bit 다양하게.

```
arm-none-eabi-gcc -g3 -O0 -c aapcs_parameters.c
```
컴파일을 해주고

```
$ objdump -t aapcs_parameters.o
aapcs_parameters.o:     file format elf32-little
SYMBOL TABLE:
...
00000000 g     F .text  0000007c caller
0000007c g     F .text  00000034 param_four
000000b0 g     F .text  00000034 param_eight
```
Object 파일에서 각 함수의 offset을 보면, 0x0부터 0xb0+0x34까지 필요한 코드가 있다는 걸 확인.

```
$ arm-none-eabi-objdump --start-address 0x00000000 --stop-address $((0x000000b0+0x00000034)) -dS aapcs_parameters.o

aapcs_parameters.o:     file format elf32-littlearm


Disassembly of section .text:

00000000 <caller>:

void param_four (uint8_t one, uint16_t two, uint32_t three, uint32_t four);
void param_eight(uint8_t one, uint16_t two, uint32_t three, uint32_t four,
                 uint8_t five, uint16_t six, uint32_t seven, uint32_t eight);

void caller() {
   0:   e92d4800        push    {fp, lr}
   4:   e28db004        add     fp, sp, #4
   8:   e24dd010        sub     sp, sp, #16
  param_four (0xaa, 0xbbaa, 0xccbbaa, 0xddccbbaa);
   c:   e59f3050        ldr     r3, [pc, #80]   ; 64 <caller+0x64>
  10:   e59f2050        ldr     r2, [pc, #80]   ; 68 <caller+0x68>
  14:   e59f1050        ldr     r1, [pc, #80]   ; 6c <caller+0x6c>
  18:   e3a000aa        mov     r0, #170        ; 0xaa
  1c:   ebfffffe        bl      7c <param_four>
  param_eight(0xaa, 0xbbaa, 0xccbbaa, 0xddccbbaa,
  20:   e59f3048        ldr     r3, [pc, #72]   ; 70 <caller+0x70>
  24:   e58d300c        str     r3, [sp, #12]
  28:   e59f3044        ldr     r3, [pc, #68]   ; 74 <caller+0x74>
  2c:   e58d3008        str     r3, [sp, #8]
  30:   e59f3040        ldr     r3, [pc, #64]   ; 78 <caller+0x78>
  34:   e58d3004        str     r3, [sp, #4]
  38:   e3a03011        mov     r3, #17
  3c:   e58d3000        str     r3, [sp]
  40:   e59f301c        ldr     r3, [pc, #28]   ; 64 <caller+0x64>
  44:   e59f201c        ldr     r2, [pc, #28]   ; 68 <caller+0x68>
  48:   e59f101c        ldr     r1, [pc, #28]   ; 6c <caller+0x6c>
  4c:   e3a000aa        mov     r0, #170        ; 0xaa
  50:   ebfffffe        bl      b0 <param_eight>
              0x11, 0x2211, 0x332211, 0x44332211);
}
  54:   e1a00000        nop                     ; (mov r0, r0)
  58:   e24bd004        sub     sp, fp, #4
  5c:   e8bd4800        pop     {fp, lr}
  60:   e12fff1e        bx      lr
  64:   ddccbbaa        .word   0xddccbbaa
  68:   00ccbbaa        .word   0x00ccbbaa
  6c:   0000bbaa        .word   0x0000bbaa
  70:   44332211        .word   0x44332211
  74:   00332211        .word   0x00332211
  78:   00002211        .word   0x00002211

0000007c <param_four>:
```

먼저 `0c:`부터가 `param_four()`를 호출하는 과정. `0c:`에서 네번째 파라미터를 r3에 넣어주고, 밑에서 r2, r1에 각각 숫자를 넣어준다. 넣어줄 숫자는 instruction과 같이 있지 않고, pc로 부터 얼마나 떨어지 offset에 있는 값을 가지고 온다. 아마 instruction은 4bytes로 다 마추려고 그런듯. `18:`에서는 마지막으로 첫 번째 파라미터(r0)을 `0xaa`로 설정해 주는데 여기서는 offset을 사용 안하고 0xaa를 바로 설정 해준다. instruction에 1byte 정도는 여유가 있나보다.

두번째 `param_eight()` 한수 호출에서 `40:`에서 `4c`는 `param_four()`와 같고. 5~6번째 파라미터를 전달하는 방식만 조금 다르다. `20:`을 보면 r3에다가 8번째 인자로 사용할 값을 `70:`에서 가져오고, r3을 다시 sp+12에 저장을 해준다. 비슷한 방법으로 7,6,5 번째 파라미터도 stack에 올려준다.

```
void param_four(uint8_t one, uint16_t two, uint32_t three, uint32_t four) {
  7c:   e52db004        push    {fp}            ; (str fp, [sp, #-4]!)
  80:   e28db000        add     fp, sp, #0
  84:   e24dd014        sub     sp, sp, #20
  88:   e50b200c        str     r2, [fp, #-12]
  8c:   e50b3010        str     r3, [fp, #-16]
  90:   e1a03000        mov     r3, r0
  94:   e54b3005        strb    r3, [fp, #-5]
  98:   e1a03001        mov     r3, r1
  9c:   e14b30b8        strh    r3, [fp, #-8]
  return;
  a0:   e1a00000        nop                     ; (mov r0, r0)
}
  a4:   e28bd000        add     sp, fp, #0
  a8:   e49db004        pop     {fp}            ; (ldr fp, [sp], #4)
  ac:   e12fff1e        bx      lr
```
그럼 받아온 파라미터는 어떻게 사용할까? 레지스터로 받아온 값은 그냥 register로 사용하면 되고, stack에 받아온 값은 stack을 읽어 들여서 사용하면 되겠지만 레지스터로 받아온 값은 다시 스텍에 넣어준다. 예제를 컴파일 할 때 `-O0`을 해줘서 그런 것도 있고, 진짜 유용한 일을 하는 함수였다면 r0-r3을 진짜 유용하게 사용하기 위해 파라미터로 묶여있지 않게 하기 위함.

```
000000b0 <param_eight>:

void param_eight(uint8_t one, uint16_t two, uint32_t three, uint32_t four,
                 uint8_t five, uint16_t six, uint32_t seven, uint32_t eight) {
  b0:   e52db004        push    {fp}            ; (str fp, [sp, #-4]!)
  b4:   e28db000        add     fp, sp, #0
  b8:   e24dd014        sub     sp, sp, #20
  bc:   e50b200c        str     r2, [fp, #-12]
  c0:   e50b3010        str     r3, [fp, #-16]
  c4:   e1a03000        mov     r3, r0
  c8:   e54b3005        strb    r3, [fp, #-5]
  cc:   e1a03001        mov     r3, r1
  d0:   e14b30b8        strh    r3, [fp, #-8]
  return;
  d4:   e1a00000        nop                     ; (mov r0, r0)
}
  d8:   e28bd000        add     sp, fp, #0
  dc:   e49db004        pop     {fp}            ; (ldr fp, [sp], #4)
  e0:   e12fff1e        bx      lr
``
원래 스텍으로 전달된 파라미터들은 이미 스텍에 있으니까 다시 스텍에 넣을 필요가 없다.
