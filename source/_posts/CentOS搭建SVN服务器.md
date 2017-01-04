---
title: CentOS搭建SVN服务器
category: "tool"
date: 2016/11/20
---

```
# yum安装
yum -y install subversion

# 创建版本库
mkdir -p /home/svn/repos
svnadmin create /home/svn/repos

# 设置用户名及密码
vim /home/svn/repos/passwd

[user]
user = user
user1 = user1
user2 = user2

# 设置用户分组及权限
vim /home/svn/repos/authz

[groups]
admin = user
java = user1,user2

[repos:/]
@admin = rw
* =

[repos:/javaweb]
@admin = rw
@java = rw

# 设置访问权限
vim /home/svn/repos/svnserve.conf
[general]
anon-access = none
auth-access = write

# 开放访问端口
iptables -I INPUT 4 -p tcp -m state --state NEW -m tcp --dport 3690 -j ACCEPT
service iptables save

# 启动svn
svnserve -d -r /home/svn/

# 查看pid
ps -ef | grep svnserve

# 关闭svn
kill -9 ${pid}

# 删除版本库
rm -rf /home/svn/repos/
```
