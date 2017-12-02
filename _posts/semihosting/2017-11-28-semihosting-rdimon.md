---
layout: post
title: 'Semihosting으로 printf() 사용하기'
date:   2017-11-28 06:24 +0900
tags: stm32 semihosting rdimon
---
STM32 개발 보드를 사용하면 st-link를 이용해서 디버거를 물릴 수 있어서 디버깅하는데 아쉬움은 없지만. 가끔은 그냥 `printf()`를 이용해서 특정 위치에서 특정 값을 출력하는 것이 더 편할 때도 있다. 구글을 찾다 보니 그렇게 Standard Input, Output 등을 Debugger로 넘겨 주는 걸 Semihosting이라고 부르는 것 같다. 그리고 Semihosting을 하는 방법이 잘 설명된 블로그가 있었다. ([출처](#출처))

먼저 "LDFLAG"에서 기존 "--specs=<...>"을 "--specs=rdimon.specs"로 바꿔주고, "-lnosys"를 지우고 "-lrdimon"을 넣어 준다.

다음으로 `main()`이 정의된 소스 파일에 `extern void initialise_monitor_handles(void);`를 추가하고, `main()` 함수 첫 줄에 `initialise_monitor_handles();`를 호출해준다.

이렇게 하면 `printf()`의 출력값이 debugger로 전달된다. 주의할 점은, OpenOCD에서 `initialise_monitor_handles()` 함수가 호출되기 전에 "arm semihosting enable"을 해줘야 한다. gdb에서는 "monitor arm semihosting enable"을 하면 된다.

```
...
(gdb) monitor arm semihosting on
semihosting is enabled
(gdb)
...
```

`printf()`의 출력물이 OpenOCD가 실행 중인 터미널에 아래와 같이 출력된다.
```
...
target halted due to debug-request, current mode: Thread
xPSR: 0x01000000 pc: 0x08001528 msp: 0x20020000, semihosting
Hello, Semihosting.
Hello, Semihosting.
Hello, Semihosting.
Hello, Semihosting.
...
```

# 출처
<http://bgamari.github.io/posts/2014-10-31-semihosting.html>

