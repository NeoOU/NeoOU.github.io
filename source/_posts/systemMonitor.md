# 系统环境配置
## 配置信息
- CPU：一核
- 内存：1024MB
- 操作系统：CentOS 6.5 64位
- 带宽: 1Mbps

## 系统环境
- 预装[微柳网络V1.5镜像](https://market.aliyun.com/products/55528001/jxsc000238.html?spm=5176.ecsPrepay.image.selectFromMarketplace.52usUH)
- JDK1.8.0_77
- tomcat7.0.69
- mysql5.6.30

# tomcat启动参数及记录
## 启动参数
### 2017-2-6
- /usr/java/jdk1.8.0_77/bin/java
- -Djava.util.logging.config.file=/usr/local/tomcat/conf/logging.properties
- -Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager
- -Djava.security.egd=file:/dev/./urandom -server -Xms256m -Xmx498m
- -Djdk.tls.ephemeralDHKeySize=2048
- -Dfile.encoding=UTF8
- -Dsun.jnu.encoding=UTF8
- -Djava.library.path=/usr/local/apr/lib
- -Djava.endorsed.dirs=/usr/local/tomcat/endorsed
- -classpath /usr/local/tomcat/bin/bootstrap.jar:/usr/local/tomcat/bin/tomcat-juli.jar
- -Dcatalina.base=/usr/local/tomcat
- -Dcatalina.home=/usr/local/tomcat
- -Djava.io.tmpdir=/usr/local/tomcat/temp
- org.apache.catalina.startup.Bootstrap start

## 宕机记录
- 2017-2-6 20:15

## 重启记录
- 2017-2-6 20:15 宕机重启
- 2017-2-7 13:48 业务重启

## 异常记录
- 2017-2-7 22:17 系统负载>1,98.0%wa


# 系统主要参数日志
## 2017-2-7
### 2017-2-7 16:54
- free -m
```
              total       used       free     shared    buffers     cached
Mem:           996        935         60          0          0         16
-/+ buffers/cache:        918         77
Swap:         1023        132        891
```
- ps aux | grep tomcat
```
USER   PID    %CPU   %MEM     VSZ       RSS     TTY   STAT    START   TIME
www   13472    0.5   31.8   2554216   325384     ?     Sl     13:48   1:09
```

### 2017-2-7 22:17
- free -m
```
              total       used       free     shared    buffers     cached
Mem:           996        935         60          0          0         11
-/+ buffers/cache:        923         73
Swap:         1023        130        893
```

- ps aux | grep tomcat
```
USER   PID    %CPU   %MEM     VSZ       RSS     TTY   STAT    START   TIME
www   13472    0.3   32.0   2556264   327160     ?     Sl     13:48   1:40
```

- top
```
top - 22:25:56 up 62 days,  7:25,  4 users,  load average: 0.99, 0.81, 0.53
Tasks:  93 total,   2 running,  91 sleeping,   0 stopped,   0 zombie
Cpu(s):  1.7%us,  0.3%sy,  0.0%ni,  0.0%id, 98.0%wa,  0.0%hi,  0.0%si,  0.0%st
Mem:   1020276k total,   956832k used,    63444k free,      304k buffers
Swap:  1048568k total,   134064k used,   914504k free,     9784k cached
```

- iostat -x

 ```
avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           0.42    0.00    0.16    0.74    0.00   98.67
Device:         rrqm/s   wrqm/s     r/s     w/s   rsec/s   wsec/s avgrq-sz avgqu-sz   await  svctm  %util
vda               0.19     0.73    0.60    0.56    35.01    10.25    39.12     0.00    1.56   0.66   0.08
```
