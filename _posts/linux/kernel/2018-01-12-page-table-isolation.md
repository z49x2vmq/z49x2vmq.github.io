---
layout: post
title:  'Meltdown 때문에 변경된 것'
date:   2018-01-12 06:50 +0900
tags: linux kernel meltdown security
---

Meltdown 때문에 Application Privilege Level(Ring 3)에서 돌고있는 코드가 Kernel Level(Ring 0)의 메모리 영역에 있는 데이터에 접근할 수 있다고 한다.

# 가상 메모리 환경
![Page Table Simplified]({{ "/assets/images/PageTable.svg" | absolute_url }})<br>
*Virtual-Physical Memory Mapping for Processes*

OS에서 각 Process는 자기만의 Virtual Memory Address 영역을 가지고 있다. 위 그림에서 Process 1000번의 D 메모리 페이지와 Process 2000의 C 메모리 페이지가 같은 주소값을 가지고 있지만 실제로는 다른 Physical 영역에 메핑이 되있다.

각각의 Process들의 Virtual Memory Address가 Physical Memory로 매핑이되는 정보를 가지고 있는게 Page Table이다. 각각의 Process들은 고유의 Page Table 정보를 가지고 있고, Context Switch가 발생할 때 CPU의 CR3 Register에 대상 Process의 Page Table이 있는 주소를 넣어 주면 Memory 관련 연산에서 해당 Page Table 정보를 이용하게 된다.

커널 메모리 영역은 약간 Shared Memory 처럼 매핑이 된다. 커널에서 사용하는 메모리 영역인 A와 B 페이지는 모든 프로세스에 공통적으로 메핑이 되어 있다. Linux에서는 모든 User 영역의 Process들이 생성이 되면 Kernel 영역도 모두 매핑이 된다. 

그렇다면 C언어 같은 걸로 코딩을 해서 커널 영역 주소를 포인터에 넣어주면 그쪽 데이터에 접근이 가능한가? 그렇지 않다. Page Table은 단순히 VMA와 PMA의 매핑정보만 가지고 있는게 아니라, 해당 주소에 접근하려면 필요한 Privilege 정보를 같이 가지고 있다. Application Level에서 실행되는 코드가 Kerenl Level에 매핑된 주소를 접근하면 에러가 발생하게된다.

## System Call
Application Level에서 실행되던 코드가 System Call 같은게 실행되면 Kernel Level로 뛰게 된다. Linux에서는 CPU "SYSCALL" Instruction이 실행되면 entry_SYSCALL_64라고 레이블된 주소로 점프하면서 Kernel Level Mode로 변경된다.

# Meltdown
앞에서 처럼 원래는 Application Level에서 Kernel에 매핑된 주소에 접근이 안되는게 정상인다. 어떻게 하는지는 몰라도 Meltdown라고 불리는 취약점을 이용하면 그게 가능하다고한다.

## Meltdown 대응 방법
![Page Table Remedy]({{ "/assets/images/PageTable_Meltdown.svg" | absolute_url }})<br>
*Additional Page Table for User*

보통은 Process의 VMA를 생성할 때 커널 영역까지 다 포함을 하지만, Meltdown 취약점 때문에 Application Mode일때는 최소한의 Kernel 영역만 매핑을 하게된다. 최소한이라 하면 SYSCALL, Interrupt 등이 발생했을때 최초로 실행되는 커널 코드가 있는 주소 영역이 되겠다. 그리고 그런 커널 영역의 코드가 시작될때 모든 커널 영역이 매핑되있는 Page Table로 슬적 바꿔주게되면 나머지 Kernel 영역 주소들도 사용이 가능해진다.

그리고 Kernel에서 Application으로 돌아 갈때 다시 Application영역용 Page Table로 CR3를 돌려 놓으면 Application Level에서는 커널 영역 매핑이 없어지게 된다.

그림에서 처럼 Application 영역에서 사용하는 Page Table에서는 커널 영역의 Page A만 매핑을 하고 있다. 만약에 SYSCALL 같은거 때문에 Kernel 영역의 코드가 실행되면 CR3 Register를 Kernel 용 Page Table을 포인팅하게 바꿔서 Page B와 같이 나머지 Page들도 다 매핑을 시켜주게 된다.

## Kernel Code
실제로 최근에 업데이트된 Kernel Code의 entry_SYSCALL_64부분을 보면 아래와 같이 "SWITCH_TO_KERNEL_CR3"라는 내용이 있다.
```
+	SWITCH_TO_KERNEL_CR3 scratch_reg=%rdi	/* to kernel CR3 */
```

# 성능 영향
Application Code와 Kernel Code 간 전환이 빈번한 경우 성능 영향이 크고, 그렇지 않은 경우에는 비교적 영향이 적다고 한다. 
