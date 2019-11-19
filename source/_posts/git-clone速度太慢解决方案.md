---
title: git clone速度太慢解决方案
date: 2019-11-19 11:18:27
tags:
    - GitHub
categories:
    - Git 
---
适用各种操作系统，本次测试于ubuntu，下载速度从二十几k提高到二百多k

1、查找域名对应的ip地址，并修改hosts文件

```shell
nslookup github.global.ssl.fastly.Net
nslookup github.com
```

将下列内容加入 /etc/hosts文件中

```shell
151.101.76.249 http://global-ssl.fastly.net
192.30.255.113 http://github.com  #此处112还是113根据自己的情况调整？
```

2、 刷新DNS缓存
linux：

```shell
sudo /etc/init.d/networking restart
```

windows：

```shell
ipconfig /flushdns
```

macOS:

```shell
sudo killall -HUP mDNSResponder
```

参考：
http://www.linuxidc.com/Linux/2017-10/148116.htm

http://blog.csdn.net/blgmh/article/details/74531982