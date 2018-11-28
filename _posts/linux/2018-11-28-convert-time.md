---
layout: post
title:  'Converting Timezone from Linux Command Line'
date:   2018-11-29 06:50 +0900
tags: timestamp 
---

Timestamp conversion is a simple math but always so confusing. Googling
"2084-01-01 12:40:10 kst to est" will almost instantly give you the result. But
this can also be easily done from command line.

From in the command prompt:
```
$ TZ='<TARGET_TZ>' date --date='TZ="<ORIG_TZ>" 2049-01-01 00:00:00'
```

For example from Seoul time to Toronto time.
```
$ TZ='America/Toronto' date --date='TZ="Asia/Seoul" 2049-01-01 00:00:00'
Thu Dec 31 10:00:00 EST 2048
```

_Warning!!!_
It is tempting to use timezone name abbreviations but using them might give you
a wrong result:
```
$ TZ='EST' date --date='TZ="KST" 2049-01-01 00:00:00'                                                              
Thu Dec 31 19:00:00 EST 2048     !!! Wrong result
```

It seems only names that can be found in "/usr/share/zoneinfo" should be used.
