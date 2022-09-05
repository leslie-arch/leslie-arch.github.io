---
title: CUDA错误集锦
date: 2021-06-20 15:38:04
tags: [ "CUDA", "devel" ]
categories:
- ["CUDA"]
- ["nvidia"]
---

我的实验环境，是拉了一个tensorflow 2.2.2的docker镜像，cuda 10.0/1/2三个版本。
```
lrwxrwxrwx 1 root root   20 Jun 20 10:54 cuda -> /usr/local/cuda-10.1
drwxr-xr-x 4 root root 4096 May  6 15:25 cuda-10.0
drwxr-xr-x 1 1000  970 4096 Jan  7 02:38 cuda-10.1
drwxr-xr-x 1 root root 4096 Jun 20 15:16 cuda-10.2
```

# 头文件
python项目中为了执行效率，会使用C/C++编写部分扩展。在编译这些扩展时，会碰到头文件缺失问题。
一般的讲，头文件问题跳不出两个原因，一是系统中确实没有安装相应的库；一是路径配置问题。

CUDA一般通过包管理器安装时，会被分成多个不同的包供下载安装，比如，在ubuntu下，
```
sudo apt search cuda
```
会出现一串长长的列表。

### 问题1  
**fatal error: crt/host_config.h: No such file or directory**

在查看已安装的CUDA包时，
```
sudo apt search cuda | grep installed

... ...
cuda-npp-dev-10-1/unknown,now 10.1.243-1 amd64 [installed,auto-removable]
cuda-npp-dev-10-2/unknown,now 10.2.89-1 amd64 [installed,auto-removable]
cuda-nvcc-10-1/unknown,now 10.1.243-1 amd64 [installed,automatic]
cuda-nvdisasm-10-1/unknown,now 10.1.243-1 amd64 [installed,automatic]
cuda-nvgraph-10-1/unknown,now 10.1.243-1 amd64 [installed,auto-removable]
cuda-nvgraph-10-2/unknown,now 10.2.89-1 amd64 [installed,auto-removable]
... ...
```
发现cuda-nvcc和nvdisasm只有10.2版本有安装。通过安装 **cuda-nvcc** ,再查找crt文件，
发现它在10.1下也有了。
```
find /usr/local/ -name "crt"
/usr/local/cuda-10.1/targets/x86_64-linux/include/crt
/usr/local/cuda-10.1/bin/crt
/usr/local/cuda-10.2/targets/x86_64-linux/include/crt
/usr/local/cuda-10.2/bin/crt
```

### 问题2

**fatal error: cublas_v2.h: No such file or directory**

ubuntu中有相关的包：
```
apt search libcublas
Sorting... Done
Full Text Search... Done
libcublas-11-0/unknown,unknown 11.2.0.252-1 amd64
  CUBLAS native runtime libraries
libcublas-11-1/unknown,unknown 11.3.0.106-1 amd64
  CUBLAS native runtime libraries
libcublas-11-2/unknown,unknown 11.4.1.1043-1 amd64
  CUBLAS native runtime libraries
libcublas-11-3/unknown,unknown 11.4.2.10064-1 amd64
  CUBLAS native runtime libraries
libcublas-dev/unknown,now 10.2.3.254-1 amd64 [installed]
  CUBLAS native dev links, headers
libcublas-dev-11-0/unknown,unknown 11.2.0.252-1 amd64
  CUBLAS native dev links, headers
libcublas-dev-11-1/unknown,unknown 11.3.0.106-1 amd64
  CUBLAS native dev links, headers
libcublas-dev-11-2/unknown,unknown 11.4.1.1043-1 amd64
  CUBLAS native dev links, headers
libcublas-dev-11-3/unknown,unknown 11.4.2.10064-1 amd64
  CUBLAS native dev links, headers
libcublas10/unknown,now 10.2.3.254-1 amd64 [installed]
  CUBLAS native runtime libraries

```
可以看到没有10.1对应的libcublas包，所以采用10.2版本CUDA了。
