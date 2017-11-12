---
layout: post
title:  "AI-Thinker ESP-01 wifi 모듈 저전력 모드"
date:   2017-11-12 21:50:01 +0900
categories: wifi esp8266
---

5분에 한번씩 센서 값을 DB에 쏴주는 장치를 만들었는데 wifi연결이 필요없는 시간이 더 길어서 전력을 좀 아꼈으면해서 ESP8266의 Deep Sleep 모드에서 전류가 얼마나 흐르나 알아보고 싶었다. Datasheet에 값은 다 나와있지만 그냥 궁금해서 직접 측정을 해봤다.

## 일반 모드
처음 전원이 들어가고 AP에 연결하는 도중에는 약 70mA 정도가 흘렀다. AP접속이 끝나고 아무것도 안하는 상태에서는 15에서 20mA를 왔다 갔다한다.

## CH_PD Pin을 이용한 절전
처음에는 CH_PD Pin을 GPIO에 연결해서 대기 상태일때 Low를 줘서 전력을 아끼는 식으로 만들었다. 아무튼 CH_PD Pin에 Low를 보내면 대기 상태로 들어가고 0.35mA 정도.

## Deep Sleep 절전
CH_PD Pin을 이용했을때 0.35mA 정도도 나쁘지 않지만 좀더 아껴보려고 Deep Sleep을 측정해보았다. Deep Sleep으로 들어가는 방법은 AT Command `AT+GSLP`를 이용하면 된다.

```
Ai-Thinker Technology Co. Ltd.

ready
WIFI CONNECTED
WIFI GOT IP
AT

OK
AT+GSLP=5000

OK
WIFI DISCONNECT
▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒
```

`AT+GSLP=<millisecond>` 커맨드는 Deep Sleep 모드로 들어갔다가 설정된 millisecond 후에 깨어나는 거라고 하는데, 깰시간이 되면 쓰레기 값이 출력되고 그냥 죽어버린 것 같다. 더 중요한 건 전류는 0.35mA로 CH_PD Pin을 이용한 것과 같다.

## 결론
Datasheet만 봐서는 잘 모르겠지만 CH_PD를 끊는 것과 Deep Sleep이 같은 것인 듯하다. Datasheet에는 Deep Sleep이 10uA라고 써있는데 좀더 알아 봐야 되겠다.
