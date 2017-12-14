---
layout: post
title:  'tcp backlog 예제'
date:   2017-12-14 06:50 +0900
tags: linux socket tcp network backlog
---

네트워크 문제를 분석하다 보니 tcp를 다시 공부해야겠다 싶다. 남이 설명한거 듣거나 읽으면 이해한 것 같은데 막상 내가하려니 자세한 내용을 잘 모르겠다.

특정 Thread에서 Connection `accept()`를 담당하는데 다른 일을 하는 Thread가 엄청 많아서 CPU가 100% 가까운 상황에다가, Incoming Connection도 너무 많으면?

# TCP Backlog
`listen()` 함수의 두번째 인자 이름이 backlog. Incoming Connection이 들어오면 보통 `accept()`를 해줘야 한다. 그런데 사실은 `accept()`를 하기전에 이미 Connection Established(Three-Way Handshake)가 된다. 그리고 더 많은 컨낵션이 한꺼번에 들어와서 다 `accept()` 해줄 시간이 없는 경우 그런 Connection은 backlog에서 받아준다.

```
  // Server Listen() 호출

  if (listen(sockfd, 5) == -1) {
    perror("listen() failed\n");
    exit(1);
  }
```

```
  // Client Connect() 호출

  conn_ret = connect(sockfd, (struct sockaddr *)&destaddr, sizeof(struct sockaddr_in));

  if (conn_ret == -1) {
    perror("Connection failed");
  }
```


man page에서 `listen()` 함수의 backlog 인자가 maximum length라고 했는데, 5를 줬을때 6개 까지 "ESTABLISHED"가 된다.

```
$ netstat -antp | grep 5000
tcp        6      0 0.0.0.0:5000            0.0.0.0:*               LISTEN      2741/poll_echo_srv
tcp       90      0 127.0.0.1:5000          127.0.0.1:45772         ESTABLISHED -
tcp       90      0 127.0.0.1:5000          127.0.0.1:45764         ESTABLISHED -
tcp       90      0 127.0.0.1:5000          127.0.0.1:45768         ESTABLISHED -
tcp        0      0 127.0.0.1:45762         127.0.0.1:5000          ESTABLISHED 2837/./poll_echo_cl
tcp        0      0 127.0.0.1:45764         127.0.0.1:5000          ESTABLISHED 2844/./poll_echo_cl
tcp       90      0 127.0.0.1:5000          127.0.0.1:45766         ESTABLISHED -
tcp       90      0 127.0.0.1:5000          127.0.0.1:45770         ESTABLISHED -
tcp        0      0 127.0.0.1:45766         127.0.0.1:5000          ESTABLISHED 2853/./poll_echo_cl
tcp       90      0 127.0.0.1:5000          127.0.0.1:45762         ESTABLISHED -
tcp        0      1 127.0.0.1:45774         127.0.0.1:5000          SYN_SENT    2877/./poll_echo_cl
tcp        0      0 127.0.0.1:45772         127.0.0.1:5000          ESTABLISHED 2862/./poll_echo_cl
tcp        0      0 127.0.0.1:45770         127.0.0.1:5000          ESTABLISHED 2860/./poll_echo_cl
tcp        0      0 127.0.0.1:45768         127.0.0.1:5000          ESTABLISHED 2858/./poll_echo_cl
```

# backlog가 꽉차면?
6개가 지나고 7번째 Client의 접속 시도는 SYN을 던지고(SYN_SENT 상태) "SYN+ACK"을 못 받아서 저러고 있다가 없어진다. 더 정확히는 최초 "SYN"을 보내고 1초 기다린 후에 재시도한다. 다시 2초를 기다렸다 또 재시도하고. 매번 재시도 할 때마다 기다리는 시간을 두 배씩 늘린다. 최초 1초. 2, 4, 8, 16, 32, 64. 64초 기다린 다음에는 그냥 포기한다.

```
$ time ./poll_echo_cli 'Echo'
Connection failed: Connection timed out

real    2m8.755s
user    0m0.000s
sys     0m0.001s
```

SYN을 계속 보내다 실패 하니까 `connect()`를 그냥 관뒀다. 1+2+4+8+16+32+64=127 ~= 2m8.755s.

![Wireshark TCP Retransmission]({{"/assets/images/wireshark_syn_re-tx.png" | absolute_url }} "Re TX"){:width="100%" border="1"}<br>
*Wireshark으로 캡쳐한 TCP Retransmission*

Wireshark으로 Retransmission을 확인해봐도 대충 기다리는 시간이 나온다.

# backlog 크기
`listen()` 함수를 호출할 때 backlog의 크기를 설정할 수 있지만, 커널에서 각 Listening Socket의 backlog 최대값을 제한하고 있다. Linux Kernel Parameter "net.core.somaxconn" 값 보다 큰값을 backlog로 지정한 경우 저절로 somaxconn 값으로 내려간다. 

```
# somaxconn 확인
# sysctl net.core.somaxconn
net.core.somaxconn = 128

# somaxconn 변경
# sysctl net.core.somaxconn=256
net.core.somaxconn = 256
```

sysctl로 쉽게 somaxconn 값을 바꿔 줄 수 있지만 이미 Listening 하고 있는 Socket의 backlog를 실시간으로 바꿔주는 건 아니다. 새로 호출되는 `listen()`에 대해서만 적용이 된다.

이미 돌고 있는 특정 서비스로 접속하는 클라이언트가 자꾸 Timeout을 맞는다면 somaxconn을 늘려주고 서비스의 Config에서도 backlog 관련 설정을 올려주고 재시작을 해줘야 한다.
