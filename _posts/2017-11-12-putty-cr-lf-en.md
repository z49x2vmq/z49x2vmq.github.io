---
layout: post
title: 'Sending "\r\n" Over Putty Serial Connection'
date:   2017-11-12 23:50:01 +0900
tags: serial putty
---

Some modules that comes with AT commands over UART requires that those AT commands end with "\r\n". It's easy to append "\r\n" to any string in programming language. But if you just want to test the module with serial terminal app sending "\r\n" is not so straight forward.

In Putty, there are several options that are related to CR("\r") or LF("\n") but none of them makes pressing enter key sends both CR and LF.

By default, Putty sends CR when Enter Key is pressed. Alternatively, `Ctrl`+`M` will also send CR. To send LF you should press `Ctrl`+`J`

* `Ctrl`+`M` : **C**arriage **R**eturn("\r")
* `Ctrl`+`J` : **L**ine **F**eed("\n")

If you want to send, for instance, "AT\r\n", you would press A -> T -> (`Ctrl`+`M` or Enter) -> `Ctrl`+`J`.
