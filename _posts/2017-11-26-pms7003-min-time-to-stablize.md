---
layout: post
title: 'PMS7003 센서 Wakeup 후 안정된 값을 얻기위한 최소 시간'
date:   2017-11-26 17:00 +0900
tags: pms7003
---

전기 절약을 위해 PMS7003 센서를 Sleep Mode로 유지하다가 측정할 때만 Wakeup 시켜서 Passive Mode로 값을 가지고 오도록 설정했다. 

PMS7003 영문버전 Datasheet을 보면 Wakeup 후 최소 30초를 기다려야 안정된 측정값을 얻을 수 있다고 쓰여 있어서 그렇게 했었다. 그런데 PM2.5였는지 PM10이였는지 잘 기억은 안 나지만 둘 중에 한 값이 항상 0으로 나왔다. 집안 공기가 깨끗하다고 좋아했는데 너무 0만 나오니까 이상해서 Wakeup 후 대기 시간을 여유 있게 60초로 바꿨더니 측정값이 나오기 시작했다.

사실 문제가 있던 당시에는 MCU Clock 설정 같은 것도 조금 잘못되어있었다. 그 때문에 STM32 `HAL_Delay(30000)` 함수가 30초보다 짧게 대기해서 그랬을 수도 있지만 중요한 건 Wakeup 하고 조금 시간이 걸리기 때문에 너무 공기가 좋다고 나온다면 대기 시간을 조금 의심해 볼 필요가 있다. 
