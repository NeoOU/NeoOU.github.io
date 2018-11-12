---
title: CentOS编译安装nginx
category:
- nginx
date: 2018/11/7
---

## 下载
```linux
[eks@local nginx]$ wget https://nginx.org/download/nginx-1.14.1.tar.gz
--2018-11-07 10:53:59--  https://nginx.org/download/nginx-1.14.1.tar.gz
Resolving nginx.org (nginx.org)... 95.211.80.227, 206.251.255.63, 2606:7100:1:69::3f, ...
Connecting to nginx.org (nginx.org)|95.211.80.227|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 1014040 (990K) [application/octet-stream]
Saving to: ‘nginx-1.14.1.tar.gz’

100%[================================================>] 1,014,040    687KB/s   in 1.4s   


[eks@local nginx]$ tar -xf nginx-1.14.1.tar.gz 
[eks@local nginx]$ ll
total 996
drwxr-xr-x 8 eks dev    4096 Nov  6 21:52 nginx-1.14.1
-rw-r--r-- 1 eks dev 1014040 Nov  6 23:26 nginx-1.14.1.tar.gz
```

## 目录结构
```linux
[eks@local nginx]$ cd nginx-1.14.1
[eks@local nginx-1.14.1]$ ll
total 748
drwxr-xr-x 6 eks dev   4096 Nov  7 10:56 auto             # 用于获取系统编译环境及系统所能提供特性
-rw-r--r-- 1 eks dev 287441 Nov  6 21:52 CHANGES          # 英文版changes
-rw-r--r-- 1 eks dev 438114 Nov  6 21:52 CHANGES.ru       # 俄文版本changes
drwxr-xr-x 2 eks dev   4096 Nov  7 10:56 conf             # 配置示例。文件夹里的配置文件会拷贝到安装目录，成为默认配置 
-rwxr-xr-x 1 eks dev   2502 Nov  6 21:52 configure        # 脚本文件，生成中间文件，为编译准备环境
drwxr-xr-x 4 eks dev   4096 Nov  7 10:56 contrib          # 第三方贡献，包括geo2nginx.pl、unicode2nginx和vim高亮
drwxr-xr-x 2 eks dev   4096 Nov  7 10:56 html             # 包含默认的50X页面和index页面的html
-rw-r--r-- 1 eks dev   1397 Nov  6 21:52 LICENSE          # BSD许可
drwxr-xr-x 2 eks dev   4096 Nov  7 10:56 man              # linux对nginx的帮助文件，用man指令打开
-rw-r--r-- 1 eks dev     49 Nov  6 21:52 README           # readme
drwxr-xr-x 9 eks dev   4096 Nov  7 10:56 src              # 源代码
```
<!--more-->
## vim的nginx代码高亮
```linux
[eks@local nginx-1.14.1]$ mkdir ~/.vim && cp -r contrib/vim/* "$_" 
```

## configure

### 参数选项
```linux
[eks@local nginx-1.14.1]$ ./configure --help

  ## with-*_module表示将该模块编译进nginx，也就是说默认不会编译该模块
  ## without-*_module表示不把该模块编译进nginx，也就是说默认会编译该模块

  --help                             print this message

  --prefix=PATH                      set installation prefix
  --sbin-path=PATH                   set nginx binary pathname
  --modules-path=PATH                set modules path
  --conf-path=PATH                   set nginx.conf pathname
  --error-log-path=PATH              set error log pathname
  --pid-path=PATH                    set nginx.pid pathname
  --lock-path=PATH                   set nginx.lock pathname

  --user=USER                        set non-privileged user for
                                     worker processes
  --group=GROUP                      set non-privileged group for
                                     worker processes

  --build=NAME                       set build name
  --builddir=DIR                     set build directory

  --with-select_module               enable select module
  --without-select_module            disable select module
  --with-poll_module                 enable poll module
  --without-poll_module              disable poll module

  --with-threads                     enable thread pool support

  --with-file-aio                    enable file AIO support

  --with-http_ssl_module             enable ngx_http_ssl_module
  --with-http_v2_module              enable ngx_http_v2_module
  --with-http_realip_module          enable ngx_http_realip_module
  --with-http_addition_module        enable ngx_http_addition_module
  --with-http_xslt_module            enable ngx_http_xslt_module
  --with-http_xslt_module=dynamic    enable dynamic ngx_http_xslt_module
  --with-http_image_filter_module    enable ngx_http_image_filter_module
  --with-http_image_filter_module=dynamic
                                     enable dynamic ngx_http_image_filter_module
  --with-http_geoip_module           enable ngx_http_geoip_module
  --with-http_geoip_module=dynamic   enable dynamic ngx_http_geoip_module
  --with-http_sub_module             enable ngx_http_sub_module
  --with-http_dav_module             enable ngx_http_dav_module
  --with-http_flv_module             enable ngx_http_flv_module
  --with-http_mp4_module             enable ngx_http_mp4_module
  --with-http_gunzip_module          enable ngx_http_gunzip_module
  --with-http_gzip_static_module     enable ngx_http_gzip_static_module
  --with-http_auth_request_module    enable ngx_http_auth_request_module
  --with-http_random_index_module    enable ngx_http_random_index_module
  --with-http_secure_link_module     enable ngx_http_secure_link_module
  --with-http_degradation_module     enable ngx_http_degradation_module
  --with-http_slice_module           enable ngx_http_slice_module
  --with-http_stub_status_module     enable ngx_http_stub_status_module

  --without-http_charset_module      disable ngx_http_charset_module
  --without-http_gzip_module         disable ngx_http_gzip_module
  --without-http_ssi_module          disable ngx_http_ssi_module
  --without-http_userid_module       disable ngx_http_userid_module
  --without-http_access_module       disable ngx_http_access_module
  --without-http_auth_basic_module   disable ngx_http_auth_basic_module
  --without-http_mirror_module       disable ngx_http_mirror_module
  --without-http_autoindex_module    disable ngx_http_autoindex_module
  --without-http_geo_module          disable ngx_http_geo_module
  --without-http_map_module          disable ngx_http_map_module
  --without-http_split_clients_module disable ngx_http_split_clients_module
  --without-http_referer_module      disable ngx_http_referer_module
  --without-http_rewrite_module      disable ngx_http_rewrite_module
  --without-http_proxy_module        disable ngx_http_proxy_module
  --without-http_fastcgi_module      disable ngx_http_fastcgi_module
  --without-http_uwsgi_module        disable ngx_http_uwsgi_module
  --without-http_scgi_module         disable ngx_http_scgi_module
  --without-http_grpc_module         disable ngx_http_grpc_module
  --without-http_memcached_module    disable ngx_http_memcached_module
  --without-http_limit_conn_module   disable ngx_http_limit_conn_module
  --without-http_limit_req_module    disable ngx_http_limit_req_module
  --without-http_empty_gif_module    disable ngx_http_empty_gif_module
  --without-http_browser_module      disable ngx_http_browser_module
  --without-http_upstream_hash_module
                                     disable ngx_http_upstream_hash_module
  --without-http_upstream_ip_hash_module
                                     disable ngx_http_upstream_ip_hash_module
  --without-http_upstream_least_conn_module
                                     disable ngx_http_upstream_least_conn_module
  --without-http_upstream_keepalive_module
                                     disable ngx_http_upstream_keepalive_module
  --without-http_upstream_zone_module
                                     disable ngx_http_upstream_zone_module

  --with-http_perl_module            enable ngx_http_perl_module
  --with-http_perl_module=dynamic    enable dynamic ngx_http_perl_module
  --with-perl_modules_path=PATH      set Perl modules path
  --with-perl=PATH                   set perl binary pathname

  --http-log-path=PATH               set http access log pathname
  --http-client-body-temp-path=PATH  set path to store
                                     http client request body temporary files
  --http-proxy-temp-path=PATH        set path to store
                                     http proxy temporary files
  --http-fastcgi-temp-path=PATH      set path to store
                                     http fastcgi temporary files
  --http-uwsgi-temp-path=PATH        set path to store
                                     http uwsgi temporary files
  --http-scgi-temp-path=PATH         set path to store
                                     http scgi temporary files

  --without-http                     disable HTTP server
  --without-http-cache               disable HTTP cache

  --with-mail                        enable POP3/IMAP4/SMTP proxy module
  --with-mail=dynamic                enable dynamic POP3/IMAP4/SMTP proxy module
  --with-mail_ssl_module             enable ngx_mail_ssl_module
  --without-mail_pop3_module         disable ngx_mail_pop3_module
  --without-mail_imap_module         disable ngx_mail_imap_module
  --without-mail_smtp_module         disable ngx_mail_smtp_module

  --with-stream                      enable TCP/UDP proxy module
  --with-stream=dynamic              enable dynamic TCP/UDP proxy module
  --with-stream_ssl_module           enable ngx_stream_ssl_module
  --with-stream_realip_module        enable ngx_stream_realip_module
  --with-stream_geoip_module         enable ngx_stream_geoip_module
  --with-stream_geoip_module=dynamic enable dynamic ngx_stream_geoip_module
  --with-stream_ssl_preread_module   enable ngx_stream_ssl_preread_module
  --without-stream_limit_conn_module disable ngx_stream_limit_conn_module
  --without-stream_access_module     disable ngx_stream_access_module
  --without-stream_geo_module        disable ngx_stream_geo_module
  --without-stream_map_module        disable ngx_stream_map_module
  --without-stream_split_clients_module
                                     disable ngx_stream_split_clients_module
  --without-stream_return_module     disable ngx_stream_return_module
  --without-stream_upstream_hash_module
                                     disable ngx_stream_upstream_hash_module
  --without-stream_upstream_least_conn_module
                                     disable ngx_stream_upstream_least_conn_module
  --without-stream_upstream_zone_module
                                     disable ngx_stream_upstream_zone_module

  --with-google_perftools_module     enable ngx_google_perftools_module
  --with-cpp_test_module             enable ngx_cpp_test_module

  --add-module=PATH                  enable external module
  --add-dynamic-module=PATH          enable dynamic external module

  --with-compat                      dynamic modules compatibility

  --with-cc=PATH                     set C compiler pathname
  --with-cpp=PATH                    set C preprocessor pathname
  --with-cc-opt=OPTIONS              set additional C compiler options
  --with-ld-opt=OPTIONS              set additional linker options
  --with-cpu-opt=CPU                 build for the specified CPU, valid values:
                                     pentium, pentiumpro, pentium3, pentium4,
                                     athlon, opteron, sparc32, sparc64, ppc64

  --without-pcre                     disable PCRE library usage
  --with-pcre                        force PCRE library usage
  --with-pcre=DIR                    set path to PCRE library sources
  --with-pcre-opt=OPTIONS            set additional build options for PCRE
  --with-pcre-jit                    build PCRE with JIT compilation support

  --with-zlib=DIR                    set path to zlib library sources
  --with-zlib-opt=OPTIONS            set additional build options for zlib
  --with-zlib-asm=CPU                use zlib assembler sources optimized
                                     for the specified CPU, valid values:
                                     pentium, pentiumpro

  --with-libatomic                   force libatomic_ops library usage
  --with-libatomic=DIR               set path to libatomic_ops library sources

  --with-openssl=DIR                 set path to OpenSSL library sources
  --with-openssl-opt=OPTIONS         set additional build options for OpenSSL

  --with-debug                       enable debug logging
```

### 安装PCRE
```linux
[eks@local nginx-1.14.1]$ ./configure --prefix=/usr/local/nginx

checking for PCRE library ... not found
checking for PCRE library in /usr/local/ ... not found
checking for PCRE library in /usr/include/pcre/ ... not found
checking for PCRE library in /usr/pkg/ ... not found
checking for PCRE library in /opt/local/ ... not found

./configure: error: the HTTP rewrite module requires the PCRE library.
You can either disable the module by using --without-http_rewrite_module
option, or install the PCRE library into the system, or build the PCRE library
statically from the source with nginx by using --with-pcre=<path> option.
```

```linux
[eks@local pcre]$ wget https://ftp.pcre.org/pub/pcre/pcre-8.42.tar.gz
[eks@local pcre]$ sudo ./configure
[eks@local pcre]$ yum list | grep gcc        # 执行PCRE的configure时会检查gcc依赖，如果系统没有安装gcc，则需要安装
[eks@local pcre]$ sudo yum install -y gcc
[eks@local pcre]$ sudo make && sudo make install
```

### 安装zlib
```linux
[eks@local nginx-1.14.1]$ ./configure --prefix=/usr/local/nginx
checking for zlib library ... not found

./configure: error: the HTTP gzip module requires the zlib library.
You can either disable the module by using --without-http_gzip_module
option, or install the zlib library into the system, or build the zlib library
statically from the source with nginx by using --with-zlib=<path> option.
```

```
[eks@local nginx-1.14.1]$ yum list |grep zlib*    
zlib.x86_64                              1.2.7-17.el7                  @anaconda
ghc-zlib.x86_64                          0.5.4.1-27.el7                epel     
ghc-zlib-devel.x86_64                    0.5.4.1-27.el7                epel     
jzlib.noarch                             1.1.1-6.el7                   base     
jzlib-demo.noarch                        1.1.1-6.el7                   base     
jzlib-javadoc.noarch                     1.1.1-6.el7                   base     
mingw32-zlib.noarch                      1.2.8-2.el7                   epel     
mingw32-zlib-static.noarch               1.2.8-2.el7                   epel     
mingw64-zlib.noarch                      1.2.8-2.el7                   epel     
mingw64-zlib-static.noarch               1.2.8-2.el7                   epel     
zlib.i686                                1.2.7-17.el7                  base     
zlib-ada.x86_64                          1.4-0.5.20120830CVS.el7       epel     
zlib-ada-devel.x86_64                    1.4-0.5.20120830CVS.el7       epel     
zlib-devel.i686                          1.2.7-17.el7                  base     
zlib-devel.x86_64                        1.2.7-17.el7                  base     
zlib-static.i686                         1.2.7-17.el7                  base     
zlib-static.x86_64                       1.2.7-17.el7                  base  
   
[eks@local nginx-1.14.1]$ sudo yum install -y zlib-devel
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
Resolving Dependencies
--> Running transaction check
---> Package zlib-devel.x86_64 0:1.2.7-17.el7 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

==========================================================================================
 Package               Arch              Version                    Repository       Size
==========================================================================================
Installing:
 zlib-devel            x86_64            1.2.7-17.el7               base             50 k

Transaction Summary
==========================================================================================
Install  1 Package

Total download size: 50 k
Installed size: 132 k
Downloading packages:
zlib-devel-1.2.7-17.el7.x86_64.rpm                                 |  50 kB  00:00:00     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : zlib-devel-1.2.7-17.el7.x86_64                                         1/1 
  Verifying  : zlib-devel-1.2.7-17.el7.x86_64                                         1/1 

Installed:
  zlib-devel.x86_64 0:1.2.7-17.el7                                                        

Complete!
```

### configure success
```
[eks@local nginx-1.14.1]$ ./configure --prefix=/usr/local/nginx

Configuration summary
  + using system PCRE library
  + OpenSSL library is not used
  + using system zlib library

  nginx path prefix: "/usr/local/nginx"
  nginx binary file: "/usr/local/nginx/sbin/nginx"
  nginx modules path: "/usr/local/nginx/modules"
  nginx configuration prefix: "/usr/local/nginx/conf"
  nginx configuration file: "/usr/local/nginx/conf/nginx.conf"
  nginx pid file: "/usr/local/nginx/logs/nginx.pid"
  nginx error log file: "/usr/local/nginx/logs/error.log"
  nginx http access log file: "/usr/local/nginx/logs/access.log"
  nginx http client request body temporary files: "client_body_temp"
  nginx http proxy temporary files: "proxy_temp"
  nginx http fastcgi temporary files: "fastcgi_temp"
  nginx http uwsgi temporary files: "uwsgi_temp"
  nginx http scgi temporary files: "scgi_temp"
```

```
[eks@local nginx-1.14.1]$ ll
total 756
-rw-r--r-- 1 eks dev      0 Nov  7 10:57 ]
drwxr-xr-x 6 eks dev   4096 Nov  7 10:56 auto
-rw-r--r-- 1 eks dev 287441 Nov  6 21:52 CHANGES
-rw-r--r-- 1 eks dev 438114 Nov  6 21:52 CHANGES.ru
drwxr-xr-x 2 eks dev   4096 Nov  7 13:05 conf
-rwxr-xr-x 1 eks dev   2502 Nov  6 21:52 configure
drwxr-xr-x 4 eks dev   4096 Nov  7 10:56 contrib
drwxr-xr-x 2 eks dev   4096 Nov  7 10:56 html
-rw-r--r-- 1 eks dev   1397 Nov  6 21:52 LICENSE
-rw-r--r-- 1 eks dev    376 Nov  7 19:07 Makefile      ## configure成功后新增的Makefile文件
drwxr-xr-x 2 eks dev   4096 Nov  7 10:56 man
drwxr-xr-x 3 eks dev   4096 Nov  7 19:11 objs          ## configure成功后新增的中间文件
-rw-r--r-- 1 eks dev     49 Nov  6 21:52 README
drwxr-xr-x 9 eks dev   4096 Nov  7 10:56 src

[eks@local nginx-1.14.1]$ cd objs/
[eks@local objs]$ ll
total 84
-rw-r--r-- 1 eks dev 17763 Nov  7 19:11 autoconf.err
-rw-r--r-- 1 eks dev 39478 Nov  7 19:11 Makefile
-rw-r--r-- 1 eks dev  6804 Nov  7 19:11 ngx_auto_config.h
-rw-r--r-- 1 eks dev   657 Nov  7 19:11 ngx_auto_headers.h
-rw-r--r-- 1 eks dev  5725 Nov  7 19:11 ngx_modules.c       ## 确定执行编译时，哪些模块会编译进nginx
drwxr-xr-x 9 eks dev  4096 Nov  7 19:11 src                 ## 源码


[eks@local objs]$ vim ngx_modules.c 

#include <ngx_config.h>
#include <ngx_core.h>



extern ngx_module_t  ngx_core_module;
extern ngx_module_t  ngx_errlog_module;
extern ngx_module_t  ngx_conf_module;
extern ngx_module_t  ngx_regex_module;
extern ngx_module_t  ngx_events_module;
extern ngx_module_t  ngx_event_core_module;
extern ngx_module_t  ngx_epoll_module;
extern ngx_module_t  ngx_http_module;
extern ngx_module_t  ngx_http_core_module;
extern ngx_module_t  ngx_http_log_module;
extern ngx_module_t  ngx_http_upstream_module;
extern ngx_module_t  ngx_http_static_module;
extern ngx_module_t  ngx_http_autoindex_module;
extern ngx_module_t  ngx_http_index_module;
extern ngx_module_t  ngx_http_mirror_module;
extern ngx_module_t  ngx_http_try_files_module;
extern ngx_module_t  ngx_http_auth_basic_module;
extern ngx_module_t  ngx_http_access_module;
extern ngx_module_t  ngx_http_limit_conn_module;
extern ngx_module_t  ngx_http_limit_req_module;
extern ngx_module_t  ngx_http_geo_module;
extern ngx_module_t  ngx_http_map_module;
extern ngx_module_t  ngx_http_split_clients_module;
extern ngx_module_t  ngx_http_referer_module;
extern ngx_module_t  ngx_http_rewrite_module;
extern ngx_module_t  ngx_http_proxy_module;
extern ngx_module_t  ngx_http_fastcgi_module;
extern ngx_module_t  ngx_http_uwsgi_module;
extern ngx_module_t  ngx_http_scgi_module;
extern ngx_module_t  ngx_http_memcached_module;
extern ngx_module_t  ngx_http_empty_gif_module;
extern ngx_module_t  ngx_http_browser_module;
extern ngx_module_t  ngx_http_upstream_hash_module;
extern ngx_module_t  ngx_http_upstream_ip_hash_module;
extern ngx_module_t  ngx_http_upstream_least_conn_module;
extern ngx_module_t  ngx_http_upstream_keepalive_module;
extern ngx_module_t  ngx_http_upstream_zone_module;
extern ngx_module_t  ngx_http_write_filter_module;
extern ngx_module_t  ngx_http_header_filter_module;
extern ngx_module_t  ngx_http_chunked_filter_module;
extern ngx_module_t  ngx_http_range_header_filter_module;
extern ngx_module_t  ngx_http_gzip_filter_module;
extern ngx_module_t  ngx_http_postpone_filter_module;
extern ngx_module_t  ngx_http_ssi_filter_module;
extern ngx_module_t  ngx_http_charset_filter_module;
extern ngx_module_t  ngx_http_userid_filter_module;
extern ngx_module_t  ngx_http_headers_filter_module;
extern ngx_module_t  ngx_http_copy_filter_module;
extern ngx_module_t  ngx_http_range_body_filter_module;
extern ngx_module_t  ngx_http_not_modified_filter_module;

ngx_module_t *ngx_modules[] = {
    &ngx_core_module,
    &ngx_errlog_module,
    &ngx_conf_module,
    &ngx_regex_module,
    &ngx_events_module,
    &ngx_event_core_module,
    &ngx_epoll_module,
    &ngx_http_module,
    &ngx_http_core_module,
    &ngx_http_log_module,
    &ngx_http_upstream_module,
    &ngx_http_static_module,
    &ngx_http_autoindex_module,
    &ngx_http_index_module,
    &ngx_http_mirror_module,
    &ngx_http_try_files_module,
    &ngx_http_auth_basic_module,
    &ngx_http_access_module,
    &ngx_http_limit_conn_module,
    &ngx_http_limit_req_module,
    &ngx_http_geo_module,
    &ngx_http_map_module,
    &ngx_http_split_clients_module,
    &ngx_http_referer_module,
    &ngx_http_rewrite_module,
    &ngx_http_proxy_module,
    &ngx_http_fastcgi_module,
    &ngx_http_uwsgi_module,
    &ngx_http_scgi_module,
    &ngx_http_memcached_module,
    &ngx_http_empty_gif_module,
    &ngx_http_browser_module,
    &ngx_http_upstream_hash_module,
    &ngx_http_upstream_ip_hash_module,
    &ngx_http_upstream_least_conn_module,
    &ngx_http_upstream_keepalive_module,
    &ngx_http_upstream_zone_module,
    &ngx_http_write_filter_module,
    &ngx_http_header_filter_module,
    &ngx_http_chunked_filter_module,
    &ngx_http_range_header_filter_module,
    &ngx_http_gzip_filter_module,
    &ngx_http_postpone_filter_module,
    &ngx_http_ssi_filter_module,
    &ngx_http_charset_filter_module,
    &ngx_http_userid_filter_module,
    &ngx_http_headers_filter_module,
    &ngx_http_copy_filter_module,
    &ngx_http_range_body_filter_module,
    &ngx_http_not_modified_filter_module,
    NULL
};

char *ngx_module_names[] = {
    "ngx_core_module",
    "ngx_errlog_module",
    "ngx_conf_module",
    "ngx_regex_module",
    "ngx_events_module",
    "ngx_event_core_module",
    "ngx_epoll_module",
    "ngx_http_module",
    "ngx_http_core_module",
    "ngx_http_log_module",
    "ngx_http_upstream_module",
    "ngx_http_static_module",
    "ngx_http_autoindex_module",
    "ngx_http_index_module",
    "ngx_http_mirror_module",
    "ngx_http_try_files_module",
    "ngx_http_auth_basic_module",
    "ngx_http_access_module",
    "ngx_http_limit_conn_module",
    "ngx_http_limit_req_module",
    "ngx_http_geo_module",
    "ngx_http_map_module",
    "ngx_http_split_clients_module",
    "ngx_http_referer_module",
    "ngx_http_rewrite_module",
    "ngx_http_proxy_module",
    "ngx_http_fastcgi_module",
    "ngx_http_uwsgi_module",
    "ngx_http_scgi_module",
    "ngx_http_memcached_module",
    "ngx_http_empty_gif_module",
    "ngx_http_browser_module",
    "ngx_http_upstream_hash_module",
    "ngx_http_upstream_ip_hash_module",
    "ngx_http_upstream_least_conn_module",
    "ngx_http_upstream_keepalive_module",
    "ngx_http_upstream_zone_module",
    "ngx_http_write_filter_module",
    "ngx_http_header_filter_module",
    "ngx_http_chunked_filter_module",
    "ngx_http_range_header_filter_module",
    "ngx_http_gzip_filter_module",
    "ngx_http_postpone_filter_module",
    "ngx_http_ssi_filter_module",
    "ngx_http_charset_filter_module",
    "ngx_http_userid_filter_module",
    "ngx_http_headers_filter_module",
    "ngx_http_copy_filter_module",
    "ngx_http_range_body_filter_module",
    "ngx_http_not_modified_filter_module",
    NULL
};

```

## make and install
```
[eks@local nginx-1.14.1]$ sudo make && sudo make install
[sudo] password for eks: 
make -f objs/Makefile
make[1]: Entering directory `/home/eks/nginx/nginx-1.14.1'
make[1]: Nothing to be done for `build'.
make[1]: Leaving directory `/home/eks/nginx/nginx-1.14.1'
make -f objs/Makefile install
make[1]: Entering directory `/home/eks/nginx/nginx-1.14.1'
test -d '/usr/local/nginx' || mkdir -p '/usr/local/nginx'
test -d '/usr/local/nginx/sbin' \
        || mkdir -p '/usr/local/nginx/sbin'
test ! -f '/usr/local/nginx/sbin/nginx' \
        || mv '/usr/local/nginx/sbin/nginx' \
                '/usr/local/nginx/sbin/nginx.old'
cp objs/nginx '/usr/local/nginx/sbin/nginx'
test -d '/usr/local/nginx/conf' \
        || mkdir -p '/usr/local/nginx/conf'
cp conf/koi-win '/usr/local/nginx/conf'
cp conf/koi-utf '/usr/local/nginx/conf'
cp conf/win-utf '/usr/local/nginx/conf'
test -f '/usr/local/nginx/conf/mime.types' \
        || cp conf/mime.types '/usr/local/nginx/conf'
cp conf/mime.types '/usr/local/nginx/conf/mime.types.default'
test -f '/usr/local/nginx/conf/fastcgi_params' \
        || cp conf/fastcgi_params '/usr/local/nginx/conf'
cp conf/fastcgi_params \
        '/usr/local/nginx/conf/fastcgi_params.default'
test -f '/usr/local/nginx/conf/fastcgi.conf' \
        || cp conf/fastcgi.conf '/usr/local/nginx/conf'
cp conf/fastcgi.conf '/usr/local/nginx/conf/fastcgi.conf.default'
test -f '/usr/local/nginx/conf/uwsgi_params' \
        || cp conf/uwsgi_params '/usr/local/nginx/conf'
cp conf/uwsgi_params \
        '/usr/local/nginx/conf/uwsgi_params.default'
test -f '/usr/local/nginx/conf/scgi_params' \
        || cp conf/scgi_params '/usr/local/nginx/conf'
cp conf/scgi_params \
        '/usr/local/nginx/conf/scgi_params.default'
test -f '/usr/local/nginx/conf/nginx.conf' \
        || cp conf/nginx.conf '/usr/local/nginx/conf/nginx.conf'
cp conf/nginx.conf '/usr/local/nginx/conf/nginx.conf.default'
test -d '/usr/local/nginx/logs' \
        || mkdir -p '/usr/local/nginx/logs'
test -d '/usr/local/nginx/logs' \
        || mkdir -p '/usr/local/nginx/logs'
test -d '/usr/local/nginx/html' \
        || cp -R html '/usr/local/nginx'
test -d '/usr/local/nginx/logs' \
        || mkdir -p '/usr/local/nginx/logs'
make[1]: Leaving directory `/home/eks/nginx/nginx-1.14.1'

[eks@local nginx]$ cd /usr/local/nginx/
[eks@local nginx]$ ll
total 16
drwxr-xr-x 2 root root 4096 Nov  7 19:11 conf    ## 配置文件夹
drwxr-xr-x 2 root root 4096 Nov  7 19:11 html    ## 默认index页面和默认50X页面
drwxr-xr-x 2 root root 4096 Nov  7 19:11 logs    ## access.log和error.log所在文件夹
drwxr-xr-x 2 root root 4096 Nov  7 19:11 sbin    ## 最主要二进制文件所在文件夹，
```

## 启动nginx

### 参数选项
```
[eks@local sbin]$ ./nginx -h
nginx version: nginx/1.14.1
Usage: nginx [-?hvVtTq] [-s signal] [-c filename] [-p prefix] [-g directives]

Options:
  -?,-h         : this help
  -v            : show version and exit
  -V            : show version and configure options then exit
  -t            : test configuration and exit
  -T            : test configuration, dump it and exit
  -q            : suppress non-error messages during configuration testing
  -s signal     : send signal to a master process: stop, quit, reopen, reload
  -p prefix     : set prefix path (default: /usr/local/nginx/)
  -c filename   : set configuration file (default: conf/nginx.conf)
  -g directives : set global directives out of configuration file
```

### 启动
```
[eks@local sbin]$ sudo ./nginx -t
[sudo] password for eks: 
nginx: the configuration file /usr/local/nginx/conf/nginx.conf syntax is ok
nginx: configuration file /usr/local/nginx/conf/nginx.conf test is successful

[eks@local sbin]$ sudo ./nginx
[sudo] password for eks: 
[eks@local sbin]$ ps aux | grep nginx
root     12247  0.0  0.0  20500   612 ?        Ss   19:49   0:00 nginx: master process ./nginx
nobody   12248  0.0  0.0  20944  1340 ?        S    19:49   0:00 nginx: worker process
eks      12256  0.0  0.0 112660   968 pts/0    R+   19:49   0:00 grep --color=auto nginx

[eks@local sbin]$ cd ..
[eks@local nginx]$ ll
total 36
drwx------ 2 nobody root 4096 Nov  7 19:43 client_body_temp
drwxr-xr-x 2 root   root 4096 Nov  7 19:11 conf
drwx------ 2 nobody root 4096 Nov  7 19:43 fastcgi_temp
drwxr-xr-x 2 root   root 4096 Nov  7 19:11 html
drwxr-xr-x 2 root   root 4096 Nov  7 19:43 logs
drwx------ 2 nobody root 4096 Nov  7 19:43 proxy_temp
drwxr-xr-x 2 root   root 4096 Nov  7 19:11 sbin
drwx------ 2 nobody root 4096 Nov  7 19:43 scgi_temp
drwx------ 2 nobody root 4096 Nov  7 19:43 uwsgi_temp

[eks@local nginx]$ cd logs/
[eks@local logs]$ ll
total 4
-rw-r--r-- 1 root root 0 Nov  7 19:43 access.log
-rw-r--r-- 1 root root 0 Nov  7 19:43 error.log
-rw-r--r-- 1 root root 6 Nov  7 19:49 nginx.pid

# 文件位置均已在configure时配置
# Configuration summary
#   + using system PCRE library
#   + OpenSSL library is not used
#   + using system zlib library
# 
#   nginx path prefix: "/usr/local/nginx"
#   nginx binary file: "/usr/local/nginx/sbin/nginx"
#   nginx modules path: "/usr/local/nginx/modules"
#   nginx configuration prefix: "/usr/local/nginx/conf"
#   nginx configuration file: "/usr/local/nginx/conf/nginx.conf"
#   nginx pid file: "/usr/local/nginx/logs/nginx.pid"
#   nginx error log file: "/usr/local/nginx/logs/error.log"
#   nginx http access log file: "/usr/local/nginx/logs/access.log"
#   nginx http client request body temporary files: "client_body_temp"
#   nginx http proxy temporary files: "proxy_temp"
#   nginx http fastcgi temporary files: "fastcgi_temp"
#   nginx http uwsgi temporary files: "uwsgi_temp"
#   nginx http scgi temporary files: "scgi_temp"
```
![welcome](/images/nginx/welcome.jpg)