# Linux启动顺序(旧)

## 1.BIOS

- BIOS是基本输入/输出系统的缩写
- 执行一些系统完整性检查
- 搜索、加载和执行启动加载器程序。
- 它在软盘、CD-ROM或硬盘中寻找启动加载器。在BIOS启动过程中，你可以按一个键（通常是F12或F2，但这取决于你的系统）来改变启动顺序。
- 一旦检测到 Boot Loader 程序并加载到内存中，BIOS 就将控制权交给它。
- 因此，简单地说，BIOS 加载并执行 MBR 启动加载器。

## 2.MBR

- MBR是主引导记录的意思。
- 它位于可启动磁盘的第一扇区。通常是/dev/hda，或/dev/sda。
- MBR的大小不超过512字节。它有三个部分：1）主引导程序inf-在前446个字节中；2）分区表inf-在接下来的64个字节中；3）MBR验证检查在最后两个字节中。
- 它包含了关于GRUB（或旧系统中的LIL-）的信息。
- 所以，简单地说，MBR加载并执行GRUB启动加载器。

## 3.GRUB

- GRUB是Grand Unified Bootloader的缩写。

- 如果你的系统上安装了多个内核镜像，你可以选择执行哪一个。

- GRUB显示一个闪屏，等待几秒钟，如果你不输入任何东西，它就加载grub配置文件中指定的默认内核镜像。

- GRUB有文件系统的知识（老的Linux加载器LIL-并不了解文件系统）。

- Grub的配置文件是/boot/grub/grub.conf（/etc/grub.conf是它的链接）。下面是CentOS的grub.conf样本。
    ```shell
    #boot=/dev/sda
    default=0
    timeout=5
    splashimage=(hd0,0)/boot/grub/splash.xpm.gz
    hiddenmenu
    title CentOS (2.6.18-194.el5PAE)
          root (hd0,0)
          kernel /boot/vmlinuz-2.6.18-194.el5PAE ro root=LABEL=/
          initrd /boot/initrd-2.6.18-194.el5PAE.img
    ```
    
    
    
- 正如你从上面的信息注意到的，它包含了内核和initrd镜像。
- 所以，简单地说，GRUB只是加载和执行内核和initrd镜像。

## 4.内核

- 挂载根文件系统，如grub.conf中的 "root="所指定。
- 内核执行/sbin/init程序
- 由于init是Linux内核执行的第一个程序，它的进程ID（PID）为1。
- initrd是初始RAM磁盘的缩写。
- initrd被内核用作临时的根文件系统，直到内核被启动，真正的根文件系统被挂载。它还包含必要的驱动程序，帮助它访问硬盘分区和其他硬件。

## 5.Init

- 查看/etc/inittab文件以决定Linux的运行级别。
- 以下是可用的运行级别
  - 0 - 停机
  - 1 - 单用户模式
  - 2 - 多用户，没有NFS
  - 3 - 完全的多用户模式
  - 4 - 未使用
  - 5 - X11
  - 6 - 重启

- Init 从 /etc/inittab 中识别默认的init级别，并使用它来加载所有适当的程序。
- 在你的系统上执行 "grep initdefault /etc/inittab "来识别默认运行级别
- 如果你想找麻烦，你可以把默认运行级别设为0或6。既然你知道0和6的意思，可能你不会去做。
- 通常情况下，你会把默认的运行级别设置为3或5。

## 6.运行级别程序

- 当Linux系统启动时，你可能会看到各种服务被启动。例如，它可能会说 "启动sendmail .... OK"。这些是运行级程序，从运行级目录中执行，由你的运行级定义。
- 根据你的默认启动级别设置，系统将从以下目录之一执行这些程序
  以下目录执行程序。
  - 运行级别0 - /etc/rc.d/rc0.d/
  - 运行级别1--/etc/rc.d/rc1.d/
  - 运行第2级--/etc/rc.d/rc2.d/
  - 运行第3级--/etc/rc.d/rc3.d/
  - 运行第4级--/etc/rc.d/rc4.d/
  - 运行第5级--/etc/rc.d/rc5.d/
  - 运行第6级--/etc/rc.d/rc6.d/
- 请注意，这些目录在/etc下也有符号链接。所以 /etc/rc0.d 被链接到/etc/rc.d/rc0.d。
- 在/etc/rc.d/rc*.d/目录下，你会看到以S和K开头的程序。
- 以S开头的程序在启动时使用。S表示启动。
- 以K开头的程序是在关机时使用的。K代表杀死。
- 在程序名称中的S和K旁边有一些数字。这些数字是程序应该被启动或杀死的顺序号。
- 例如，S12syslog是用来启动syslog deamon的，它的序列号是12。S80sendmail是启动sendmail守护进程，它的序列号是80。因此，syslog程序将在sendmail之前被启动。
