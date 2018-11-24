---
layout: post
title:  'Quick I/O Performance Check With "dd"'
date:   2018-09-25 06:50 +0900
tags: linux dd 
---

An I/O intensive applications, such as, DBMS can suffer from slow I/O performance. But unfortunately, it is not always clear that the root cause of the bad performance really lies in I/O layer. 

With `dd` command it is really easy to test I/O performance of a filesystem.

For write test you can run below `dd` command in any directory of filesystem with I/O performance problem. (For instance, data directory of DBMS)

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

# Direct I/O
According to the result above, everything seems fine(251 MB/s for write, 615 MB/s for read). But your application or database might still be suffering from slow I/O performance.

It's because above test is not a fair disk I/O test because most of the I/O was done in the filesystem cache in the memory.

For the I/O performance test we want to measure the time the entire contents are actually written on to the disk.

With `direct` flag for `dd` command it will bypass the cache and read/write directly from/to the disk.


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

# Note
If block size is not specified or is set to wrong value, the result will be very poor. It is important to set the `bs` to some multiple of physical block size.
