# 交换空间

## 什么是交换空间？

Linux中的交换空间是在物理内存（RAM）的数量已满时使用的。如果系统需要更多的内存资源，而RAM已经满了，内存中不活动的页面就会被移到交换空间。

虽然交换空间可以帮助拥有少量内存的机器，但它不应该被认为是更多内存容量的替代品。交换空间位于硬盘，它的访问时间比物理内存慢。另外在当代的容器环境下，都需要关闭交换空间，因为在操作系统看来容器实际上是一个进程，有交换空间会导致容器性能问题。

交换空间推荐配置：

| 内存大小       | 交换空间      | 如果允许休眠，建议的交换空间 |
| :------------- | :------------ | :--------------------------- |
| ⩽ 2 GB         | 2倍的内存量   | 3倍的内存量                  |
| > 2 GB - 8 GB  | 等于RAM的数量 | 2倍于RAM的数量               |
| > 8 GB - 64 GB | 至少4GB       | 1.5倍的内存量                |
| > 64 GB        | 至少4 GB      | 不建议休眠                   |

[swap（redhat.com）](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/storage_administration_guide/ch-swapspace)

Commands
dd
mkswap
swapon or swapoff

## 从现有的磁盘创建交换空间的步骤

如果你没有任何额外的磁盘，你可以在文件系统的某个地方创建一个文件，并使用该文件作为交换空间。

下面的dd命令例子在/目录下创建了一个名为 "newswap "的交换文件，其大小为1024MB（1.0GB）。

```shell
dd if=/dev/zero of=/newswap bs=1M count=1024
```

if = 从FILE而不是stdin中读取信息 
of = 写到FILE而不是stdout 
bs = 一次读和写BYTES 
count = 文件的总大小

改变交换文件的权限，以便只有root可以访问它。

```shell
chmod go-r /newswap
或者
chmod 0600 /newswap
```

使用 mkswap 命令将这个文件作为交换文件。

```shell
mkswap /newswap
```

启用新创建的newswap。

```shell
swapon /newswap
```

为了使这个交换文件在重启后也能作为交换区使用，在/etc/fstab文件中添加以下一行。

```shell
cat /etc/fstab
/newswap swap swap defaults 0 0
```


验证新创建的交换区是否可供你使用。

```shell
swapon -s
free -h
```

如果你不想重启来验证系统是否占用了 /etc/fstab 中提到的所有交换空间，你可以做如下操作，这将禁用和启用/etc/fstab中提到的所有交换分区 

中提到的所有交换分区

```shell
# 关闭
swapoff -a 
# 开启
swapon -a
```



