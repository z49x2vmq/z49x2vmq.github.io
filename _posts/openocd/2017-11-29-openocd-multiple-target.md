---
layout: post
title:  'OpenOCD로 여러보드 동시에 Debugging 하기'
date:   2017-11-29 06:50 +0900
category: openocd
tags: stm32 openocd gdb
---

---
# 관련 포스트
{% for post in site.categories.openocd %} 
+ [{{ post.title }}]({{ post.url }}) {% endfor %}

---

보드 여러 개를 동시에 디버깅하려면 openOCD도 그만큼 실행을 해줘야 하는데 옵션을 안 주고 실행을 하면 어떤 보드에 붙었는지 잘 알 수가 없다.

# Serial Number를 이용해서 특정 보드에 붙이기
```
$ lsusb
Bus 002 Device 016: ID 0483:374b STMicroelectronics ST-LINK/V2.1
Bus 002 Device 015: ID 0483:374b STMicroelectronics ST-LINK/V2.1
...
```
lsusb 커맨드로 보면 두 개가 뜬다. 어느 놈이 어느 놈인지 알아보려면 USB를 뺐다 꽂아보면서 없어졌다가 다시 나오는 놈을 확인하는 약간의 노동이 필요하다. 일단 사라졌다 나오는 놈을 찾았으면 Bus 번호랑 Device 번호를 이용해서 Serial Number를 찾을 수 있다.

```
$ lsusb -s 002:016 -v | grep iSerial
  iSerial                 3 XXXXXXXXXXXXXXXXXXXXXXX
```

이제 시리얼 번호를 이용해서 openocd를 실행할 때 특정 보드에 붙게 할 수 있다.

```
$ openocd -f <...> -c "hla_serial XXXXXXXXXXXXXXXXXXXXXXX"
```

# 여러 보드 Debugging
위에서처럼 하면 여러 보드를 동시에 디버깅할 수 있을 것 같지만 openocd에서 open하는 tcp port 3333,4444,5555들이 이미 열려있는 상태이기 때문에 다른 openocd를 실행하면 포트를 열수 없어서 정상적으로 실행되지 않는다.

그럴 때는 openocd에서 open하는 port 번호를 바꿔주면 된다.
```
$ openocd -f <...>  -c "hla_serial XXXXXXXXXXXXXXXXXXXXXXX; gdb_port 3334; tcl_port 3335; telnet_port 3336"
$ openocd -f <...>  -c "hla_serial YYYYYYYYYYYYYYYYYYYYYYY; gdb_port 4334; tcl_port 4335; telnet_port 4336"
```

# Bash script
```shell
#!/bin/bash
# vim: ts=4:sw=4:expandtab

usage() {
    echo "Usage: $0 -f <openocd_script> [-s <board_serial> | -u <usbbus:devnum>] -p <port_prefix>"

    exit 1
}

while getopts ":f:s:u:p:" opt; do
    case ${opt} in
    f)
        SCRIPT=$OPTARG
        ;;
    s)
        SERIAL=$OPTARG
        ;;
    u)
        USBDEV=$OPTARG
        ;;
    p)
        PORTPREFIX=$OPTARG
        ;;
    :)  
        echo "Invalid Option"
        usage
        ;;
    esac
done

if [ -z "$SERIAL" ]; then
    if [ -z "$USBDEV" ]; then
        echo "Both -s and -u are not specified"
        exit 1
    fi

    SERIAL=$(lsusb -s $USBDEV -v 2> /dev/null | grep iSerial | awk '{print $3}')
fi

if [ -n "$PORTPREFIX" ]; then
    re='^[0-9]+$'
    if ! [[ $PORTPREFIX =~ $re ]] ; then
        echo "Port prefix is not a number"
        usage
    fi

    GDBPORT=${PORTPREFIX}3
    TELPORT=${PORTPREFIX}4
    TCLPORT=${PORTPREFIX}6
elif [ -z "$PORTPREFIX" ]; then
    echo "Port prefix is not given"
    usage
fi

if [ -z "$SCRIPT" ]; then
    echo "Script is not given"
    usage
fi

#echo $SCRIPT
#echo $SERIAL
#echo $USBDEV
#echo $PORTPREFIX

openocd -f $SCRIPT  -c "hla_serial $SERIAL; gdb_port $GDBPORT; tcl_port $TCLPORT; telnet_port $TELPORT"
```
