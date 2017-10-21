---
title: How to Add Virtual Memory In Linux
date: 2017-09-18 00:04:48
tags:
- Ubuntu
- Linux
---


### Server environment
```
$ cat /etc/issue
  Ubuntu 14.04.5 LTS
```
<!--more-->

### View the memory

```
$ free -h
```
```
             total       used       free     shared    buffers     cached
Mem:          2.0G       243M       1.7G       424K        11M       145M
-/+ buffers/cache:        86M       1.9G
Swap:         0B          0B        0B
```
As shown, the memory is 2G.

### Use the dd command to create a file swapfile with a size of 2G：
```
$ dd if=/dev/zero of=/mnt/swapfile bs=1M count=2048
```
- "if" means input_file, "of" means output_file, "bs" means block_size, "count" means counting way. Here, I used the data block size 1M, the number of data blocks 2048, so that the allocation of space is 2G size.

### Format the swap file：
```
$ mkswap /mnt/swapfile
```

```
Setting up swapspace version 1, size = 2 GiB (2147479552 bytes)
no label, UUID=e509fbd9-f640-446d-9261-180f2bdb444c
```

### Mount the swap file：
```
$ swapon /mnt/swapfile
```

```
swapon: /mnt/swapfile: insecure permissions 0644, 0600 suggested.
```

In this way, you can see the memory size after adding 2G virtual memory, as shown in the figure, a total of 4G.
```
$ free -h
```
```
             total       used       free     shared    buffers     cached
Mem:          2.0G       243M       1.7G       424K        11M       145M
-/+ buffers/cache:        86M       1.9G
Swap:         2.0G         0B       2.0G
```

### Modify the configuration file, automatically load virtual memory after boot
```
$ sudo vim /etc/fstab
```
### Add the following command to the /etc/fstab file：
```
/mnt/swapfile swap swap defaults 0 0
```

Ok ,the tutorial has been done.

### If you use a period of time, the free memory becomes less and less, you can try to release some spare memory space
```
$ sync
$ su
$ echo 3 > /proc/sys/vm/drop_caches
```