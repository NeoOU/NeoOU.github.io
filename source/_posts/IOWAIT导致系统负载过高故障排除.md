---
title: IOWAIT导致系统负载过高故障排除
category:
- 系统运维
date: 2017/02/23
---

# 1. 系统环境
- CentOs 6.5
- 1核1GB内存

# 2. 问题描述
系统负载频繁告警<br>
![loadAverage0218](/images/IOWAIT/loadAverage.png)
![cpuUseage0218](/images/IOWAIT/cpuUseage.png)

# 3. 问题解决
## 3.1 查因
### 3.1.1 定位哪个进程引起的IOWAIT
- 安装iotop命令
```
yum install iotop
```
- 使用iotop查看IO状态
![IOTOP](/images/IOWAIT/IOTOP.png)

### 3.1.2 关于kswapd0
- Linux uses kswapd for virtual memory management such that pages that have been recently accessed are kept in memory and less active pages are paged out to disk.
- 系统每过一定时间就会唤醒kswapd进程，查看内存资源是否紧张，如果不紧张，则继续睡眠。在kswapd中，有2个阀值，pages_hige和pages_low，当空闲内存页的数量低于pages_low的时候，kswapd进程就会扫描内存并且每次释放出32个free pages，直到free page的数量到达pages_high。
- 也就是说kswapd0是有内存资源不足而唤醒的，它去扫描并释放空闲内存，期间会执行大量的换页操作，极有可能就是此进程造成的IO 100%耗尽。

## 3.2 解决方法
加1G内存<br>
![loadAverage0223](/images/IOWAIT/loadAverage0223.png)

# 参考文章
- [Troubleshooting High I/O Wait in Linux](http://bencane.com/2012/08/06/troubleshooting-high-io-wait-in-linux/)
- [quan：Exadata计算节点由kswapd0进程引起的IO使用率100%，内存不足的故障处理](http://blog.itpub.net/22878696/viewspace-1805953/)
