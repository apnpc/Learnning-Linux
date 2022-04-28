# Samba 安装

## 1 简要说明

Samba是一个Linux工具或实用程序，允许与其他操作系统共享 Linux 资源，如文件和打印机。

它的工作原理与 NFS 完全相同，但不同的是NFS在Linux或类似Unix的系统中共享，而Samba则与其他操作系统（如Windows、MAC等）共享。

例如，计算机 "A "使用Samba与计算机 "B "共享其文件系统，那么计算机 "B "将看到该共享文件系统，就像它被挂载在本地文件系统一样。

Samba 通过一个名为SMB（服务器信息块）的协议共享其文件系统，该协议是由 IBM 发明的。

另一个用于共享Samba的协议是通过微软发明的CIFS（通用互联网文件系统）和 NMB（NetBios 名称服务器）。

CIFS成为SMB的扩展，现在微软已经推出了较新的 SMB v2 和 v3 版本，这些版本在业界被广泛使用。

大多数人在使用SMB或CIFS时，谈论的都是同一件事。 两者不仅在讨论中可以互换，而且在应用中也可以互换，即使用CIFS的客户端可以与使用SMB的服务器对话，反之亦然。 为什么呢？因为CIFS是SMB的一种形式。

一步一步的安装说明

首先，请确保对你的虚拟机进行快照

## 2 安装 samba 软件包

```shell
yum install samba samba-client samba-common
```

启用允许通过防火墙的samba（仅当你有防火墙在运行）。
```shell
firewall-cmd --permanent --zone=public --add-service=samba
firewall-cmd –reload
```

要停止和禁用防火墙或iptables

```shell
systemctl stop firewalld
systemctl stop iptables
systemctl disable firewalld
systemctl disable iptables
```

创建Samba共享目录并分配权限

```shell
mkdir -p /samba/morepretzels
chmod a+rwx /samba/morepretzels
chown -R nobody:nobody /samba
```

另外，你需要改变samba共享目录的SELinux安全环境，如下所示。(只有当你启用了SELinux时)

```shell
chcon -t samba_share_t /samba/morepretzels
```

如果你想禁用SELinux，请遵循以下说明

```shell
# 检查SELinux状态
sestatus

# 修改 SELINUX=enforcing 为 SELINUX=disabled
vi /etc/selinux/config

# 或者直接执行
sed -i '/^SELINUX=/ s/=.*/=disabled/ ' /etc/selinux/config
reboot
```

修改/etc/samba/smb.conf文件，添加新的共享文件系统（确保创建一个smb.conf文件的副本）。

删除smb.conf文件中的所有内容，添加以下参数

```shell\
[global]
  workgroup = WORKGROUP 
  netbios name = centos 
  security = user
  map to guest = bad user 
  dns proxy = no
[Anonymous]
  path = /samba/morepretzels 
  browsable = yes
  writable = yes 
  guest ok = yes 
  guest only = yes 
  read only = no
```

核实设置

```shell
testparm
```

一旦软件包安装完毕，启用并启动Samba服务

```shell
systemctl enable smb
systemctl enable nmb
systemctl start smb
systemctl start nmb
```

## 3 在 Windows 客户端上安装

打开资源管理器，直接在地址栏中输入 `\\192.168.1.95` （这是我的服务器 IP，你可以通过运行 `ip a` 命令检查你的 Linux CentOS IP）

## 4 挂载在 Linux 客户端

```shell
yum -y install cifs-utils samba-client
```

创建一个挂载点目录

```shell
mkdir /mnt/sambashare
```



挂载samba共享

```shell
mount -t cifs //192.168.1.95/Anonymous /mnt/sambashare/
Entry without password
```

## 5 配置Samba 服务器

创建一个组smbgrp和用户larry，通过适当的认证访问samba服务器。

```
useradd larry
groupadd smbgrp
usermod -a -G smbgrp larry
smbpasswd -a larry
```

新的SMB密码: YOUR SAMBA PASS

重新输入新的SMB密码

重复你的Samba密码
添加用户 larry

创建一个新的共享，设置共享的权限。

```
mkdir /samba/securepretzels
chown -R larry:smbgrp /samba/securepretzels
chmod -R 0770 /samba/securepretzels
chcon -t samba_share_t /samba/securepretzels
```

编辑配置文件/etc/samba/smb.conf (先创建一个备份副本)

```
vi /etc/samba/smb.conf
```

添加以下几行

```
[Secure]
  path = /samba/securepretzels
  valid users = @smbgrp
  guest ok = no
  writable = yes
  browsable = yes
```

重起服务

```shell
systemctl restart smb
systemctl restart nmb
```

