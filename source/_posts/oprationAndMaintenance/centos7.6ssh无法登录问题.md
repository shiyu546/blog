---
title: CentOS 7.6 ssh远程登录提示permisson denied问题
date: 2024-04-22 15:09:11
categories:
- [运维,服务器运维]
tags:
- 运维
- centos
---

## 问题

有一台服务器，版本是centos7.6,发现通过ssh登录产生Permission denied, please try again的提示。通过管理平台登录到服务器桌面之后，使用systemctl status sshd查看sshd的状态，发现有下列红色的报错语句：
> error Could not get shadow infromation for root.

通过搜索得到了两个关键信息网页：

> https://access.redhat.com/solutions/46137
> https://forums.opensuse.org/t/ssh-trouble-with-shadow-information/167277

第一个链接记录了问题的描述，在centos6上，如果同时设置usePAM=no和selinux=on,则无法通过ssh连接上服务器。第二个链接记录了这两个参数的说明：

> It does not explain why sshd fails to access /etc/shadow.
>
> For the sake of archives - MicroOS defaults to SELinux in targeted enforcing mode. When UsePAM is enabled,  pam_unix module calls unix_chkpwd binary which is allowed to read /etc/shadow. Without PAM, sshd tries to read this file directly which is prohibited by SELinux for its (sshd systemd service process) context.
>
> If you start sshd within your root login session which has unconfined SELinux context, authentication succeeds.

这里解释了这两个参数的作用，所以有两种解决办法，要么开启usePAM，要么关闭selinux.

## 解决方式

下面采用关闭selinux的方式：

1. 查看selinux的开启状态：

> /usr/sbin/sestatus  -v

如果显示
> SELinux status:   enabled

表示selinux处于启用状态。

2. 临时关闭selinux:

> $: setenforce 0

3. 永久关闭selinux:

- 打开配置文件/etc/selinux/config,修改如下配置:

> $: SELINUX=disabled
