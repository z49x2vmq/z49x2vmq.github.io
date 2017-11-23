---
layout: post
title: 'aapcs : Leaf Vs. Non-Leaf Function'
date:   2017-11-17 23:50:01 +0900
tags: arm gcc gdb aapcs
description: Leaf 함수와 Non-Leaf 함수의 호출 방식 차이점
categories: AAPCS
--- 

ARM에서는 R14를 LR(Link-Register)라는 특수한 용도로 사용을 한다. 이 LR은 Branch Instruction으로 분기를 할때 Return Address를 담고 있다가 Return을 할때 사용한다. 하지만 더 이상 다른 함수를 호출하지않는 Leaf 함수라면 LR 값을 계속 가지고 있어도 상관 없지만, 다른 함수를 호출하는 함수라면 LR은 어떻게 해야 할까? 당연히 스텍에 Push를 한다.

아래와 같은 코드를 컴파일해서 보자
```c
void NO_RET_NO_ARG_NO_LEAF();
void NO_RET_NO_ARG_LEAF();

void NO_RET_NO_ARG_NO_LEAF() {
  NO_RET_NO_ARG_LEAF();
  return;
}

void NO_RET_NO_ARG_LEAF() {
  return;
}
```
간단한 코드다. `NO_RET_NO_ARG_NO_LEAF()`에서 `NO_RET_NO_ARG_LEAF()`을 호출한다. `NO_RET_NO_ARG_LEAF()`은 이름처럼 Leaf Function이라 아무것도 호출하지 않는다.

```
$ arm-none-eabi-gcc -g3 -O0 -c leaf_no_leaf.c
```
컴파일을 할 때는 `-g3`으로 디버그 info를 최대한 많이 생성해주고, `-O0`으로 방금 작성한 아무짝에도 쓸모없는 코드가 진짜 아무짝에도 쓸모없는 취급을 받아 버려지지 안도록 해준다. 아무짝에도 쓸모 없지만 예제로써 쓸모있으니까.
```
$ arm-none-eabi-objdump -t leaf_no_leaf.o 

leaf_no_leaf.o:     file format elf32-littlearm

SYMBOL TABLE:
...
00000000 g     F .text  0000001c NO_RET_NO_ARG_NO_LEAF
0000001c g     F .text  00000018 NO_RET_NO_ARG_LEAF
```
먼저 생성된 Symbol 정보를 파악하고, Start Address와 Size를 기반으로 Disassemble(`-d`)을 해주자. C 소스와 같이 보면 이해하기 편하니까 `-S` 옵션도 같이.
```
$ arm-none-eabi-objdump -M reg-names-apcs -dS --start-address 0x00000000 --stop-address $((0x00000000 + 0x0000001c + 0x00000018)) leaf_no_leaf.o

leaf_no_leaf.o:     file format elf32-littlearm


Disassembly of section .text:

00000000 <NO_RET_NO_ARG_NO_LEAF>:
void NO_RET_NO_ARG_NO_LEAF();
void NO_RET_NO_ARG_LEAF();

void NO_RET_NO_ARG_NO_LEAF() {
   0:   e92d4800        push    {fp, lr}
   4:   e28db004        add     fp, sp, #4
  NO_RET_NO_ARG_LEAF();
   8:   ebfffffe        bl      1c <NO_RET_NO_ARG_LEAF>
  return;
   c:   e1a00000        nop                     ; (mov r0, r0)
}
  10:   e24bd004        sub     sp, fp, #4
  14:   e8bd4800        pop     {fp, lr}
  18:   e12fff1e        bx      lr

0000001c <NO_RET_NO_ARG_LEAF>:

void NO_RET_NO_ARG_LEAF() {
  1c:   e52db004        push    {fp}            ; (str fp, [sp, #-4]!)
  20:   e28db000        add     fp, sp, #0
  return;
  24:   e1a00000        nop                     ; (mov r0, r0)
}
  28:   e28bd000        add     sp, fp, #0
  2c:   e49db004        pop     {fp}            ; (ldr fp, [sp], #4)
  30:   e12fff1e        bx      lr
```
`0:`에서는 LR의 값이 다음 함수 호출때 세로운 값으로 뭉개지기 때문에 FP와 같이 LR을 Stack에 Push해준다. 하지만 `1c:`를 보면 FP만 Push를 해준다.

AAPCS:<br>
{% for item in site.aapcs %} <a href="{{ item.url }}">{{ item.title }}</a><br> {% endfor %}
