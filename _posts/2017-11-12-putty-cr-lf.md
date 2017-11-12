---
layout: default
title:  "Putty로 Serial Port 접속시 \r\n 날리기"
date:   2017-11-12 23:50:01 +0900
categories: serial putty
---

UART로 소통하는 모듈은 사용전에 PC에서 터미널 프로그램을 이용해서 미리 접속해서 기능을 알아 볼 수있다. 이번에 구입한 ESP-01 모듈의 AT Command를 시험해보려고 했는데, 모든 커맨드가 \<CR>\<LF>로 끝나야 한다는 제약이 있다.

여러 터미널 프로그램으로 접속을 시도했지만 엔터를 쳤을때 \<CR>\<LF>를 동시에 보내는 터미널은 없었다. 그러다 우연히 putty에서 혹시나 하는 마음에 ctrl+j를 눌렀더니 \<LF>날아 갔다. 

Putty에서는 먼저 엔터키를 치면 \<CR>이 날아가고 바로 ctrl+j를 눌러서 \<LF>를 날려주면 된다.
