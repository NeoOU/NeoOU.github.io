---
title: CentOS用户管理
category:
- linux
- Operation & Maintenance
date: 2018/11/6
---
 
### 指令
```linux
[root@local ~]# useradd --help
Usage: useradd [options] LOGIN 
       useradd -D
       useradd -D [options]

Options:
  -b, --base-dir BASE_DIR       base directory for the home directory of the
                                new account
  -c, --comment COMMENT         GECOS field of the new account
  -d, --home-dir HOME_DIR       home directory of the new account
  -D, --defaults                print or change default useradd configuration
  -e, --expiredate EXPIRE_DATE  expiration date of the new account
  -f, --inactive INACTIVE       password inactivity period of the new account
  -g, --gid GROUP               name or ID of the primary group of the new
                                account
  -G, --groups GROUPS           list of supplementary groups of the new
                                account
  -h, --help                    display this help message and exit
  -k, --skel SKEL_DIR           use this alternative skeleton directory
  -K, --key KEY=VALUE           override /etc/login.defs defaults
  -l, --no-log-init             do not add the user to the lastlog and
                                faillog databases
  -m, --create-home             create the user's home directory
  -M, --no-create-home          do not create the user's home directory
  -N, --no-user-group           do not create a group with the same name as
                                the user
  -o, --non-unique              allow to create users with duplicate
                                (non-unique) UID
  -p, --password PASSWORD       encrypted password of the new account
  -r, --system                  create a system account
  -R, --root CHROOT_DIR         directory to chroot into
  -s, --shell SHELL             login shell of the new account
  -u, --uid UID                 user ID of the new account
  -U, --user-group              create a group with the same name as the user
  -Z, --selinux-user SEUSER     use a specific SEUSER for the SELinux user mapping
```
<!--more-->
```linux
[root@local ~]# usermod --help
Usage: usermod [options] LOGIN

Options:
  -c, --comment COMMENT         new value of the GECOS field
  -d, --home HOME_DIR           new home directory for the user account
  -e, --expiredate EXPIRE_DATE  set account expiration date to EXPIRE_DATE
  -f, --inactive INACTIVE       set password inactive after expiration
                                to INACTIVE
  -g, --gid GROUP               force use GROUP as new primary group
  -G, --groups GROUPS           new list of supplementary GROUPS
  -a, --append                  append the user to the supplemental GROUPS
                                mentioned by the -G option without removing
                                him/her from other groups
  -h, --help                    display this help message and exit
  -l, --login NEW_LOGIN         new value of the login name
  -L, --lock                    lock the user account
  -m, --move-home               move contents of the home directory to the
                                new location (use only with -d)
  -o, --non-unique              allow using duplicate (non-unique) UID
  -p, --password PASSWORD       use encrypted password for the new password
  -R, --root CHROOT_DIR         directory to chroot into
  -s, --shell SHELL             new login shell for the user account
  -u, --uid UID                 new UID for the user account
  -U, --unlock                  unlock the user account
  -Z, --selinux-user SEUSER     new SELinux user mapping for the user account
```

```
[root@local ~]# userdel --help
Usage: userdel [options] LOGIN

Options:
  -f, --force                   force some actions that would fail otherwise
                                e.g. removal of user still logged in
                                or files, even if not owned by the user
  -h, --help                    display this help message and exit
  -r, --remove                  remove home directory and mail spool
  -R, --root CHROOT_DIR         directory to chroot into
  -Z, --selinux-user            remove any SELinux user mapping for the user
```

```
[root@local ~]# groupadd --help
Usage: groupadd [options] GROUP

Options:
  -f, --force                   exit successfully if the group already exists,
                                and cancel -g if the GID is already used
  -g, --gid GID                 use GID for the new group
  -h, --help                    display this help message and exit
  -K, --key KEY=VALUE           override /etc/login.defs defaults
  -o, --non-unique              allow to create groups with duplicate
                                (non-unique) GID
  -p, --password PASSWORD       use this encrypted password for the new group
  -r, --system                  create a system account
```

```
[root@local ~]# groupmod --help
Usage: groupmod [options] GROUP

Options:
  -g, --gid GID                 change the group ID to GID
  -h, --help                    display this help message and exit
  -n, --new-name NEW_GROUP      change the name to NEW_GROUP
  -o, --non-unique              allow to use a duplicate (non-unique) GID
  -p, --password PASSWORD       change the password to this (encrypted)
                                PASSWORD
  -R, --root CHROOT_DIR         directory to chroot into
```

```
[root@local ~]# groupdel --help
Usage: groupdel [options] GROUP

Options:
  -h, --help                    display this help message and exit
  -R, --root CHROOT_DIR         directory to chroot into
```

### 用户、分组管理

```
# 新增用户分组
[root@local ~]# groupadd dev

# 查看分组信息
# 组名:口令:组id:组内用户列表
# "组名"是用户组的名称，组名不能重复。
# "口令"字段存放的是用户组加密后的口令字。一般Linux 系统的用户组都没有口令，即这个字段一般为空，或者是*，或者是X。
# "组id"是一个整数，被系统内部用来标识组。
# "组内用户列表"是属于这个组的所有用户的列表，不同用户之间用英文逗号(,)分隔。这个用户组可能是用户的主组，也可能是附加组。
[root@local ~]# cat /etc/group | grep dev
dev:x:1000:
```

```$xslt
# 新增用户并设置密码
[root@local ~]# useradd -g dev eks
[root@local ~]# passwd eks
Changing password for user eks.
New password: 
BAD PASSWORD: The password is shorter than 7 characters
Retype new password: 
passwd: all authentication tokens updated successfully.

# 查看用户信息
# 用户名:口令:用户id:组id:注释性描述:主目录:登录Shell
[root@local ~]# cat /etc/passwd | grep eks
eks:x:1000:1000::/home/eks:/bin/bash
[root@local ~]# cat /etc/group |grep eks
dev:x:1000:eks

# 配置root权限
# 确认/etc/sudoers文件中的wheel分组的权限打开(没有用#注释掉)
[root@local ~]# cat /etc/sudoers
## Allows people in group wheel to run all commands
%wheel  ALL=(ALL)       ALL

[root@local ~]# cat /etc/group | grep wheel
wheel:x:10:

# 把用户添加wheel分组
[root@local ~]# usermod -a -G wheel eks
[root@local ~]# cat /etc/group | grep eks
wheel:x:10:eks
dev:x:1000:eks

# 禁止用户ssh登陆
[root@local ~]# usermod -s /sbin/nologin eks
[root@local ~]# cat /etc/passwd | grep eks
eks:x:1000:1000::/home/eks:/sbin/nologin
```
