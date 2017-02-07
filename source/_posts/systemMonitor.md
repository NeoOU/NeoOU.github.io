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
