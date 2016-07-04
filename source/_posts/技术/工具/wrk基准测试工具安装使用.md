---
title: wrk基准测试工具安装使用
thumbnail: 'http://7xiovs.com1.z0.glb.clouddn.com/my-pic-P60608-195617.jpg?imageView2/1/w/800/h/600/q/75|watermark/2/text/Qnkg6ZOB5rGk/font/5a6L5L2T/fontsize/500/fill/I0VGRUZFRg==/dissolve/100/gravity/SouthEast/dx/30/dy/30'
date: 2016-03-04 09:06:33
categories:
	- 技术
	- 工具
tags:
	- 基准测试
	- wrk
keywords: 基准测试,wrk
description:
---




## git

https://github.com/wg/wrk

git clone https://github.com/wg/wrk.git

## 安装

在makefile中33行
LDIR     = deps/luajit/src
LIBS    := -lluajit $(LIBS)
CFLAGS  += -I$(LDIR)
LDFLAGS += -L$(LDIR)

下面添加：

LDFLAGS  +=  -L/usr/local/opt/openssl/lib
CFLAGS += -I/usr/local/opt/openssl/include


make

## 基本使用

Basic Usage

  wrk -t12 -c400 -d30s http://127.0.0.1:8080/index.html

  This runs a benchmark for 30 seconds, using 12 threads, and keeping
  400 HTTP connections open.

  Output:

```
  Running 30s test @ http://127.0.0.1:8080/index.html
    12 threads and 400 connections
    Thread Stats   Avg      Stdev     Max   +/- Stdev
      Latency   635.91us    0.89ms  12.92ms   93.69%
      Req/Sec    56.20k     8.07k   62.00k    86.54%
    22464657 requests in 30.00s, 17.76GB read
  Requests/sec: 748868.53
  Transfer/sec:    606.33MB
```

## 参数说明

```
$ wrk
Usage: wrk <options> <url>                            
  Options:                                            
    -c, --connections <N>  Connections to keep open   
    -d, --duration    <T>  Duration of test           
    -t, --threads     <N>  Number of threads to use   
                                                      
    -s, --script      <S>  Load Lua script file       
    -H, --header      <H>  Add header to request      
        --latency          Print latency statistics   
        --timeout     <T>  Socket/request timeout     
    -v, --version          Print version details      
                                                      
  Numeric arguments may include a SI unit (1k, 1M, 1G)
  Time arguments may include a time unit (2s, 2m, 2h)
  
```


