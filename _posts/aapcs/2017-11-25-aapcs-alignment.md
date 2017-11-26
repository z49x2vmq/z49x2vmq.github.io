---
layout: post
title: 'aapcs : Alignment'
date:   2017-11-25 14:03 +0900
tags: arm gcc gdb aapcs
categories: AAPCS
---

대부분의 CPU는 메모리 접근 효율을 위해 Align 된 주소로 접근을 한다. 왜 그렇게 해야 효율적인지는 모르겠지만 그렇다고 한다.

Align 된 주소란 대부분 액세스하려는 Data의 크기의 배수인 주소를 말한다. 예를 들면 uint32_t 처럼 4바이트 데이터는 0, 4, 8, 12, 16,... 같은 주소가 Align 된 주소다. 1, 5, 7과 같은 주소는 4의 배수가 아니므로 Align 된 주소가 아니다. 

# Sample Code
```c
#include <stdint.h>
#include <stdio.h>

struct ALIGN01 {
  uint32_t four_bytes1;
  uint32_t four_bytes2;
  uint16_t two_bytes1;
  uint8_t one_byte1;
  uint8_t one_byte2;
};

struct ALIGN02 {
  uint8_t one_byte1;
  uint32_t four_bytes1;
  uint8_t one_byte2;
  uint32_t four_bytes2;
  uint16_t two_bytes1;
};

struct ALIGN01 align01 = { 0xa4a3a2a1, 0xb4b3b2b1, 0xc2c1, 0xd1, 0xe1};
struct ALIGN02 align02 = { 0xd1, 0xa4a3a2a1, 0xe1, 0xb4b3b2b1, 0xc2c1};

int main(int argc, char **argv) {

  printf("Size of ALIGN01 : %d\n", sizeof(struct ALIGN01));
  printf("Size of ALIGN02 : %d\n", sizeof(struct ALIGN02));

  return 0;
}
```
예제에 두 가지 구조체가 있다. ALING01, ALIGN02 둘 다 uint32_t 두 개, uint16_t 한 개, uint8_t 한 개. 합하면 각 12 Bytes를 사용한다. 

# Compile 및 실행
```
$ gcc -std=c11 -g3 -O0 -o alignment alignment.c
$ ./alignment
Size of ALIGN01 : 12
Size of ALIGN02 : 20
```
두 개의 구조체 모두 맴버들 크기의 합이 12 바이트인데. `sizeof()` 결과를 보면 ALIGN01은 12바이트로 문제가 없어 보이는데, ALIGN02는 20바이트를 차지하고 있다. 

# Alignment 확인
`struct ALIGN01 align01`과 `struct ALIGN02 align02`의 값이 메모리상에 실제로 어떻게 들어가 있는지 확인.
```
$ objdump -F -t alignment -j .data -s

alignment:     file format elf32-littlearm

SYMBOL TABLE:
00020628 l    d  .data  00000000              .data
00020628  w      .data  00000000              data_start
00020630 g     O .data  0000000c              align01
00020650 g       .data  00000000              _edata
00020628 g       .data  00000000              __data_start
0002062c g     O .data  00000000              .hidden __dso_handle
0002063c g     O .data  00000014              align02
00020650 g     O .data  00000000              .hidden __TMC_END__


Contents of section .data:  (Starting at file offset: 0x628)
 20628 00000000 00000000 a1a2a3a4 b1b2b3b4  ................
 20638 c1c2d1e1 d1000000 a1a2a3a4 e1000000  ................
 20648 b1b2b3b4 c1c20000                    ........
```
먼저 `00000000 00000000` 다음에 `a1a2a3a4`가 `align01.four_bytes1`의 값이다. (c source code에서는 a4a3a2a1이였는데 여기서 a1a2a3a4로 보여지는 이유는 Little Endian이고 objdump에서 byte 순서 그대로 출력하기 때문이다.)

`align01` 멤버들의 값은 순서대로 빈틈없이 빽빽하게 나열된 것을 확인할 수 있다. (`a1a2a3a4` `b1b2b3b4` `c1c2` `d1` `e1`)

하지만 `d1000000`로 시작하는 `align02`의 값들은 듬성듬성 나열되어있다. (`d1` `000000` `a1a2a3a4` `e1` `000000` `b1b2b3b4` `c1c2` `000000`) 먼저 `d1` 값은 `align02.one_byte1`의 값이다. 그리고 다음 순서인 `four_bytes1` 값을 메모리에 넣으려니 4의 배수 주소에 넣어야 하므로 3바이트를 건너뛰고 뒤쪽에 4 bytes align이 맞는 주소에 값을 넣어주게 된다. `e1` 다음에 3 bytes를 건너뛴 것도 같은 이유에서다.

그럼 `c1c2` 까지 딱 끊기면 18 bytes면 되는데 왜 마지막에 `0000`이 있어서 20 bytes로 만드나? `struct`의 alignement는 멤버들의 alignment중 가장 큰 값을 따르고, `struct`의 사이즈는 alignment의 배수여야 한다는 제약이 있어서 그렇다. `struct ALIGN02`의 alignment는 멤버중 aligment가 가장 큰 uint32_t를 따라서 4가 되고. 4의 배수중에서 모든 멤버를 다 커버할 수 있는 크기는 20이 된다.
