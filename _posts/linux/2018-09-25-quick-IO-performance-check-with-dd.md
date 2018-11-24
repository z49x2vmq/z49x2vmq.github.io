---
layout: post
title:  'Quick I/O Performance Check With "dd"'
date:   2018-09-25 06:50 +0900
tags: linux dd 
---

An I/O intensive application, such as DBMS, can suffer from slow I/O
performance. But unfortunately, it is not always clear that the root cause of
the bad performance is in I/O layer. 

With `dd` command it is really easy to test I/O performance of a file system.
With `dd` you can only perform sequential I/O, but for most of time this will
be sufficient.

For write performance test you can execute `dd` command as below within any
directory of file system with I/O performance problem.

```
dd if=/dev/zero of=test bs=1024 count=10240
10240+0 records in
10240+0 records out
10485760 bytes (10 MB, 10 MiB) copied, 0.0418238 s, 251 MB/s
```

Similarly, read test can be done as below.

```
dd if=test of=/dev/zero bs=1024 count=10240
10240+0 records in
10240+0 records out
10485760 bytes (10 MB, 10 MiB) copied, 0.0170566 s, 615 MB/s
```

I/O throughput of 251 MB/s write and 615 MB/s read doesn't look so bad, does
it? 

## Buffered I/O

According to the results from above, everything seems fine(251 MB/s for write,
615 MB/s for read). But your application's log file or performance statistics
data might still complaining I/O performance.

Why do we get this? This is because above test is not a fair disk I/O test
because most of the I/O was done in the file system buffer(or cache) in memory.

When you complain I/O performance to system administrator, it is not very
uncommon to get this kind of result back from them.

## Direct I/O

As oppose to buffered I/O, direct I/O bypasses file system buffer. If your
application decide to maintain its own file buffer, OS file system buffer can
be disabled and save some memory. And for this type of application, measuring
buffered I/O performance is meaningless.

## Direct I/O with "dd"

With `direct` flag you can tell `dd` to perform a direct I/O. This way, we can
check pure disk performance.

```
# Write test with direct flag
dd if=/dev/zero of=test bs=1024 count=10240 oflag=direct
10240+0 records in
10240+0 records out
10485760 bytes (10 MB, 10 MiB) copied, 2.33167 s, 4.5 MB/s

# Read test with direct flag
dd if=test of=/dev/zero bs=1024 count=10240 iflag=direct
10240+0 records in
10240+0 records out
10485760 bytes (10 MB, 10 MiB) copied, 2.58185 s, 4.1 MB/s
```

Now the I/O performance bottleneck looks very clear.
