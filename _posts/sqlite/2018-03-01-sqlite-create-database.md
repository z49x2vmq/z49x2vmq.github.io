---
layout: post
title: 'sqlite db 만들기'
date:   2018-03-01 06:00:00 +0900
tags: sqlite
---

quick start guide에 `sqlite3 test.db` 만 해도 database가 생성 된다고 되있는데 안된다.

```
$ sqlite3 newdb.db
SQLite version 3.20.1 2017-08-24 16:21:36
Enter ".help" for usage hints.
sqlite> .quit
$ ls
$
```

좀 해보니 바로 `.quit`하고 나오면 안되고 아무거나 뭐라도 좀 치고 나와야 되는 것 같다.

```
$ sqlite3 newdb.db
SQLite version 3.20.1 2017-08-24 16:21:36
Enter ".help" for usage hints.
sqlite> .database
main: .../newdb.db
sqlite> .quit
$ ls
newdb.db
$
```
