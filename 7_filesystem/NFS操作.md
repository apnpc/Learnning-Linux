# 配置NFS服务器

安装NFS软件包

```
yum install nfs-utils libnfsidmap          (很可能已经安装了)
```

软件包安装完毕，启用并启动NFS服务
```
systemctl enable rpcbind 

systemctl enable nfs-server

systemctl start rpcbind, nfs-server, rpc-statd, nfs-idmapd
```

创建NFS共享目录并分配权限
```
mkdir /mypretzels

chmod a+rwx /mypretzels
```

修改/etc/exports文件，添加新的共享文件系统
```
  /mypretzels 192.168.12.7(rw,sync,no_root_squash) = for only 1 host 

/mypretzels *(rw,sync,no_root_squash) = for everyone
```

导出NFS文件系统
```
exportfs -rv
```

停止并禁用Firewalld 
```
systemctl stop firewalld

systemctl disable firewalld
```

NFS客户端配置的步骤
安装NFS软件包

```
yum install nfs-utils rpcbind
```

软件包安装完毕，启用并启动rpcbind服务
```
systemctl start rpcbind
```

确保Firewalld或iptables停止（如果正在运行）。
```
ps –ef | egrep “firewall|iptable” 
```

显示来自NFS服务器的挂载

```
showmount -e 192.168.1.5  (NFS Server IP)
```

创建一个挂载点 
```
mkdir /mnt/kramer
```

挂载NFS文件系统
```
mount 192.168.1.5:/mypretzels /mnt/kramer
```

验证安装的文件系统
```
df –h
```

要卸载
```
umount /mnt/kramer
```

