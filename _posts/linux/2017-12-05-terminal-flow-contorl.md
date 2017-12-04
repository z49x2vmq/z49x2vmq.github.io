---
layout: post
title:  'Shell 이 멈춰 버렸을 때'
date:   2017-12-05 06:50 +0900
tags: linux terminal bash
---

Linux에서 터미널이 가끔 멈춰 버리는 일이 발생한다. 프로그램이 실행되다 행걸린 것도 아니고. 그냥 Prompt만 떠있을 뿐인데 아무 입력이 되지 않는다.

보통 실수로 `Ctrl`+`S`(XOFF)를 눌렀을 때 그런일이 생기고, `Ctrl`+`Q`(XON)를 누르면 원래대로 돌아오고 멈췄을 때 눌렀던 키들이 다 먹혀있는 걸 확인 할 수 있다.
