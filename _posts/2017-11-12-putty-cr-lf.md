---
layout: post
title: 'Putty로 Serial 접속시 "\r\n" 날리기'
date:   2017-11-12 23:50:01 +0900
categories: serial putty
image: /assests/wire.jpg
---

UART로 소통하는 모듈은 사용전에 PC에서 터미널 프로그램을 이용해서 미리 접속해서 기능을 알아 볼 수있다. 이번에 구입한 ESP-01 모듈의 AT Command를 시험해보려고 했는데, 모든 커맨드가 \<CR>\<LF>로 끝나야 한다는 제약이 있다.

여러 터미널 프로그램으로 접속을 시도했지만 엔터를 쳤을때 \<CR>\<LF>를 동시에 보내는 터미널은 없었다. 그러다 우연히 putty에서 혹시나 하는 마음에 ctrl+j를 눌렀더니 \<LF>날아 갔다. 

Putty에서는 먼저 엔터키를 치면 \<CR>이 날아가고 바로 ctrl+j를 눌러서 \<LF>를 날려주면 된다. 엔터키로 \<CR>을 날릴수도 있지만 ctrl+m을 눌러도 \<CR>이 날아간다.

- CR : Carriage Return
- LF : Line Feed

옛날 영화에서 타자기 치다가 끝까지 가면 막대기 잡고 다시 앞으로 밀어주는게 CR. 그리고나서 다음 줄로 이동하기 위해 종이를 위로 조금 올려 주는게 LF.
