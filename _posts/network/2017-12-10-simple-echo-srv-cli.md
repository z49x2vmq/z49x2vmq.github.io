---
layout: post
title:  '간단한 socket 상태 변화 예제'
date:   2017-12-10 06:50 +0900
tags: linux socket tcp network
---

```c
// simple_srv.c

#include <sys/types.h>
#include <sys/socket.h>

#include <netinet/in.h>
#include <netinet/ip.h>

#include <stdlib.h>
#include <stdio.h>
#include <errno.h>
#include <string.h>
#include <unistd.h>

int main(int argc, char **argv) {
  int sockfd;
  struct sockaddr_in sockaddr;

  sockaddr.sin_family = AF_INET;
  sockaddr.sin_port = htons(atoi(argv[1]));
  sockaddr.sin_addr.s_addr = INADDR_ANY;

  sockfd = socket(AF_INET, SOCK_STREAM, 0);

  if (bind(sockfd, (struct sockaddr *)&sockaddr, sizeof(struct sockaddr_in)) == -1) {
    fprintf(stderr, "bind() error: %s\n", strerror(errno));
    close(sockfd);
    exit(1);
  }

  if (listen(sockfd, 1) == -1) {
    fprintf(stderr, "listen() error: %s\n", strerror(errno));

    close(sockfd);
    exit(1);
  }

  int clifd;
  struct sockaddr_in cliaddr;
  socklen_t socklen = sizeof(struct sockaddr_in);

  clifd = accept(sockfd, (struct sockaddr *)&cliaddr, &socklen);

  if (clifd == -1) {
    fprintf(stderr, "accept() error: %s\n", strerror(errno));

    close(clifd);
    exit(1);
  }

#define BUFFER_SIZE 1024
  char buf[BUFFER_SIZE];

  int recvsize;
  recvsize = recv(clifd, buf, BUFFER_SIZE - 1, 0);

  if (recvsize == -1) {
    fprintf(stderr, "recv() error: %s\n", strerror(errno));
    close(clifd);
    close(sockfd);
    exit(1);
  }

  buf[recvsize] = '\0';
  fprintf(stdout, "%s", buf);
  fflush(stdout);

  int sendsize;
  sendsize = send(clifd, buf, recvsize, 0);

  if (sendsize == -1) {
    fprintf(stderr, "send() error: %s\n", strerror(errno));
    close(clifd);
    close(sockfd);
    exit(1);
  }

  close(clifd);
  close(sockfd);

  return 0;
}
```

```c
// simple_cli.c

#include <sys/types.h>
#include <sys/socket.h>

#include <netinet/in.h>
#include <netinet/ip.h>
#include <arpa/inet.h>

#include <stdlib.h>
#include <stdio.h>
#include <errno.h>
#include <string.h>
#include <unistd.h>

int main(int argc, char **argv) {
  int sockfd;
  struct sockaddr_in sockaddr;

  sockfd = socket(AF_INET, SOCK_STREAM, 0);

  sockaddr.sin_family = AF_INET;
  sockaddr.sin_port = htons((short)atoi(argv[2]));
  sockaddr.sin_addr.s_addr = inet_addr(argv[1]);

  socklen_t socklen = sizeof(struct sockaddr_in);

  if (connect(sockfd, (struct sockaddr *)&sockaddr, socklen) == -1) {
    fprintf(stderr, strerror(errno));

    close(sockfd);
    exit(1);
  }

#define BUFFER_SIZE 1024
  int buf[BUFFER_SIZE];

  int sendsize;
  sendsize = send(sockfd, argv[3], strlen(argv[3]), 0);

  if (sendsize == -1) {
    fprintf(stderr, "send() error: %s\n", strerror(errno));
    close(sockfd);
    exit(0);
  }

  int recvsize;

  recvsize = recv(sockfd, buf, BUFFER_SIZE - 1, 0);

  if (recvsize == -1) {
    fprintf(stderr, "recv() error: %s\n", strerror(errno));
    close(sockfd);
    exit(1);
  }

  buf[recvsize] = '\0';
  fprintf(stdout, "%s", buf);

  close(sockfd);

  return 0;
}
```

실행 예)
```
Server
$ ./simple_srv 7777

Client
$ ./simple_cli 127.0.0.1 7777 'Echo This!!!'

Server
$ ./simple_srv 7777
Echo This!!!

Client
$ ./simple_cli 127.0.0.1 7777 'Echo This!!!'
Echo This!!!
```
먼저 simple_srv를 실행하면 Command Argument로 지정한 포트를 열고 접속을 기다린다. simple_cli로 접속 Destination IP,Port를 주고 전송할 메세지를 주고 실행하면 Destination으로 메세지를 전송한다.

simple_srv는 전송 받은 메세지를 자신의 stdout에 한번 뿌려주고, 받은 내용을 그대로 되돌려준다. simple_cli에서는 되돌려 받은 메세지를 자신의 stdout에 출력하고 종료한다.

```
$ gdb simple_srv -ex 'set args 7777'
(gdb) break main
(gdb) run
(gdb) advance 29
(gdb) n

bash
$ netstat -anpt --listen | grep 7777
tcp        0      0 0.0.0.0:7777            0.0.0.0:*               LISTEN      10248/simple_srv
```
simple_srv와 simple_cli를 gdb에 물려서 socket이 생성되고 닫히는 과정을 관찰 할 수 있다.

먼저 simple_srv를 `listen()` 함수 호출을 하고 멈춘 상태에서 netstat 커맨드로 확인을 해보면, 7777포트가 열린 것을 확인 할 수 있다.

```
$ gdb simple_cli -ex 'set args 127.0.0.1 7777 "Echo This"'
(gdb) break main
(gdb) run
(gdb) advance 26
(gdb) n

bash
$ netstat -anpt --listen | grep 7777
tcp        1      0 0.0.0.0:7777            0.0.0.0:*               LISTEN      10248/simple_srv    
tcp        0      0 127.0.0.1:7777          127.0.0.1:36228         ESTABLISHED -                   
tcp        0      0 127.0.0.1:36228         127.0.0.1:7777          ESTABLISHED 10481/simple_cli
```
simple_cli를 `connect()` 함수 까지만 실행하고 멈춰있는 상태에서 netstat을 해보면 "ESTABLISHED" 상태인 Connection이 두 개가 있다. 하나는 딱 봐도 simple_cli에서 사용하는 Connection인데, 하나는 "-"으로 나와있다.

Linux 2.2 부터는 `accept()` 하기전에 "ESTABLISHED"가 된다고 한다. "tcp" 다음 열에 "1"이 아직 `accept()` 되지 않은 Connection의 개수. 

아무튼 그래서 simple_cli입장에서는 Connection이 맺어진 상태인거고. simple_srv입장에서는 아직 `accept()`를 안했기 때문에 주인이 없는 Connection처럼 보인다.

`accept()`를 하면 어떻게 되는지 보자.

```
simple_srv gdb 세션
(gdb) advance 40
(gdb) n

bash
 netstat -anpt --listen | grep 7777
tcp        0      0 0.0.0.0:7777            0.0.0.0:*               LISTEN      10248/simple_srv    
tcp        0      0 127.0.0.1:7777          127.0.0.1:36228         ESTABLISHED 10248/simple_srv    
tcp        0      0 127.0.0.1:36228         127.0.0.1:7777          ESTABLISHED 10481/simple_cli
```

`accept()`를 했더니 "1"이 "0"으로 바뀌었다. 그리고 주인 없이 "-"로 보이던 Connection이 "10248/simple_srv"로 바뀌었다.

이제는 simple_cli에서 메세지를 던져보자.
```
simple_clie gdb 세션. 마지막으로 실행된 connect() 다음이 바로 send()라서 그냥 "n" + <Enter> 치면 된다.
(gdb) n

bash
$ netstat -anpt --listen | grep 7777
tcp        0      0 0.0.0.0:7777            0.0.0.0:*               LISTEN      10248/simple_srv    
tcp        9      0 127.0.0.1:7777          127.0.0.1:36228         ESTABLISHED 10248/simple_srv    
tcp        0      0 127.0.0.1:36228         127.0.0.1:7777          ESTABLISHED 10481/simple_cli
```

simple_cli에서 `send()`를 하면 "Echo This"가 simple_srv로 전송이된다. netstat 결과 처럼 simple_srv의 Connection에 "Recv-Q"에 9바이트가 들어가 있다. Network로 데이터가 전송되도 받아줘야하는 프로그램에서 바로 받아 줄 수 없기 때문에 "Recv-Q"라는 메모리에 일단 넣어 놓게 된다. 보통은 데이터가 들어오면 바로바로 받아 주기 때문에 "Recv-Q"가 "0"이 아니라면 받아주는 Process가 정상이 아닐 가능성이 크다.

simple_srv 같은 경우도 debugger에 물려서 `recv()` 함수 호출이 안되서 "Recv-Q"가 증가한거다. gdb에서 `recv()`까지 하면 어떻게 되는지 함 보자.

```
simple_srv gdb 세션.
(gdb) advance 53
(gdb) n
(gdb) printf "%s\n", buf
Echo This

bash
$ netstat -anpt --listen | grep 7777
tcp        0      0 0.0.0.0:7777            0.0.0.0:*               LISTEN      10248/simple_srv    
tcp        0      0 127.0.0.1:7777          127.0.0.1:36228         ESTABLISHED 10248/simple_srv    
tcp        0      0 127.0.0.1:36228         127.0.0.1:7777          ESTABLISHED 10481/simple_cli
```

`recv()` 함수가 실행되면, "Recv-Q"가 다시 0으로 돌아온다. gdb에서 buf에 들어온 내용물을 보면 simple_cli에서 보내준 "Echo This"가 있다.

이제는 simple_srv가 simple_cli한테 자신이 받은 메세지를 그대로 돌려줘보자.

```
simple_srv gdb 세션
(gdb) advance 67
(gdb) n

bash
$ netstat -anpt --listen | grep 7777
tcp        0      0 0.0.0.0:7777            0.0.0.0:*               LISTEN      10248/simple_srv    
tcp        0      0 127.0.0.1:7777          127.0.0.1:36228         ESTABLISHED 10248/simple_srv    
tcp        9      0 127.0.0.1:36228         127.0.0.1:7777          ESTABLISHED 10481/simple_cli 

simple_cli gdb 세션
(gdb) advance 47
(gdb) n

bash
$ netstat -anpt --listen | grep 7777
tcp        0      0 0.0.0.0:7777            0.0.0.0:*               LISTEN      10248/simple_srv    
tcp        0      0 127.0.0.1:7777          127.0.0.1:36228         ESTABLISHED 10248/simple_srv    
tcp        0      0 127.0.0.1:36228         127.0.0.1:7777          ESTABLISHED 10481/simple_cli
```
이번에는 simple_cli의 Recv-Q에 9바이트가 들어있다. 그리고 simple_cli에서 `recv()`를 해주면 없어진다.

마지막으로 통신이 끝났으면 Connection을 닫아 줘야한다.

```
simple_srv gdb 세션
(gdb) advance 76

simple_cli gdb 세션
(gdb) advance 58
(gdb) n

bash
# netstat -antp --listen | grep 7777
tcp        0      0 0.0.0.0:7777            0.0.0.0:*               LISTEN      17247/simple_srv    
tcp        0      0 127.0.0.1:37586         127.0.0.1:7777          FIN_WAIT2   -                   
tcp        1      0 127.0.0.1:7777          127.0.0.1:37586         CLOSE_WAIT  17247/simple_srv

simple_srv gdb 세션
(gdb) n

bash
# netstat -antp --listen | grep 7777
tcp        0      0 0.0.0.0:7777            0.0.0.0:*               LISTEN      17247/simple_srv    
tcp        0      0 127.0.0.1:37586         127.0.0.1:7777          TIME_WAIT   -
```

먼저 simple_cli에서 `close()`를 호출하면 simple_cli의 Connection이 FIN_WAIT2 상태가 된다. (그전에 FIN_WAIT1 상태는 순식간에 지나가서 보기 힘들다.)

# TCP State 변화 과정
TCP State 변화 과정을 간략하게 그려보면 아래와 같다. 더 자세하고 정확한 내용은 Google(TCP State, TCP FSM, etc).
## Connection 과정
![TCP Connection Establishment]({{ "/assets/images/connection.svg" | absolute_url }}){:width="70%" border="1" margin="auto"}<br>
*TCP Connection Establishment 과정*

simple_cli에서 `connect()` 함수를 호출하면 Three-Way Handshake를 한다. SYN과 ACK이 있는데, SYN을 보내면 받는 쪽에서 ACK을 보내준다. SYN을 보냈을 때 ACK이 돌아 오지 않으면 "아 네트워크 어딘가에서 연결이 안되는 구나"라는 걸 알 수 있다. 먼저 simple_cli에서 SYN을 보내면 simple_srv에서 SYN+ACK을 보낸다. simple_srv가 ACK만 보내지 않고 SYN+ACK을 보내는 이유는 ACK만 보낼 경우 내가 보낸 ACK이 진짜 simple_cli까지 도달했는지 알 방법이 없기 때문이다. 그래서 SYN+ACK을 보내고 simple_cli에서 ACK이 돌아온 경우 양방향 통신이 가능하다고 확신 할 수 있다.

## Close 과정
![TCP Closing steps]({{ "/assets/images/closing.svg" | absolute_url }}){:width="70%" border="1"}<br>
*TCP Connection Closing 과정*

그림은 simple_cli가 먼저 close()를 시작한 경우지만 simple_srv가 먼저 했다면 그냥 좌우만 바꾸면 된다. close()가 동시에 시작된 경우는 Google.
