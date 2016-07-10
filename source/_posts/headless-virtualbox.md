---
title: 虚拟机的正确打开方式
date: 2016-07-10 12:53:25
tags:
- virtual machine
---

由于我自从11年开始就一直在使用 \*nix 的操作系统，加上也不玩网络编程，所以虚拟机对我来说一直没怎么有用。上一次使用虚拟机还是在上学期做密码学作业的时候整了个 ubuntu 虚拟机来检查代码兼容性。昨天学 Puppet 的时候发现它们的示例上用的 `apache` 模块在 mac 上没法用，故而又拿出了虚拟机。

显然我们的虚拟机不需要装图形界面，于是我就安装了 ubuntu 最新的服务器版本。

服务器只有命令行界面，带来了两个不便：1. 它的终端输出没法回滚；2. 有时我需要查看虚拟机上的 web server 的网页。怎么解决呢？只要让虚拟机的 ssh 端口和 web 端口可被宿主访问，我就可以用宿主上强大的 iTerm 和 chrome 来访问咯。

<!-- more -->

搜了一下，果然 virtualbox 提供这个功能，大致设置过程如下：

### 1. 确认 Host-only Network

打开 virtualbox 自身的设置选项，在 Network 标签页上可以看到我们已经有了两个默认设置好的宿主网络配置，如果没有，就自己新建一个就好了。

![](/images/virtualbox1-global-settings.png)

### 2. 虚拟机建立 Host-only Network Adapter

打开虚拟机的设置面板，在 Network 标签下，新建一个 Adapter，选择为 Host-only Adapter 即可。

![](/images/virtualbox2-local-setting.png)

注意勾上 Enable Network Adapter 。

### 3. 设置网卡自动挂载

此时启动虚拟机，然后用 `ifconfig` 查看，看看是否已经有一个新的被分配了局域网地址的适配器选项，如果已经有了，你就可以从宿主端正常访问这个局域网地址了。如果没有，我们就用 `ifconfig -a` 列出所有适配器，我们会发现一个新的没启用的适配器。我们用 `ifup <enpBlahblah>` 把它启用。但是我们还需要让它每次开机自动挂载啊。这里根据不同的 Linux 发行版可能略有不同，在 Ubuntu 下，是修改 `/etc/network/interfaces` ，添加一个和 "primary network interface" 类似的配置即可，比如我的 host-only 的适配器名字是 `enp0s8`:

```shell
# The primary network interface
auto enp0s3
iface enp0s3 inet dhcp

# Host-only interface
auto enp0s8
iface enp0s8 inet dhcp
```

### 4. 后台使用

通过 ifconfig 我发现了这台虚拟机的地址为 `192.168.59.103` ，为了让虚拟机接受 ssh 连接，记得要安装一个 ssh 的服务器。在 ubuntu ，是：

```shell
sudo apt-get install openssh-server
```

一般安装好了 ssh 服务就自动打开了。我们可以通过 `service --status-all` 来检验。同时我们可以通过 `service ssh start|stop|restart` 来控制它的开启关闭。

此时我们可以把虚拟机关闭了。然后在虚拟机启动的按钮角落，有一个小小的下拉按钮，通过它，我们就可以选择 headless 的模式启动，这样它就不会建立任何窗口了。

![](/images/virtualbox3-start.png)

在 virtualbox 的预览窗口里看到系统启动完毕之后，我们就可以从 iTerm 里 ssh 进去咯：

```shell
╭─YangLiu@my13  ~/git/blog ‹2.3.1› ‹master*›
╰─$ ssh god@192.168.59.103
god@192.168.59.103's password:
Welcome to Ubuntu 15.10 (GNU/Linux 4.2.0-16-generic x86_64)

 * Documentation:  https://help.ubuntu.com/

New release '16.04 LTS' available.
Run 'do-release-upgrade' to upgrade to it.

hello, world!
Last login: Sun Jul 10 13:18:25 2016 from 192.168.59.3
╭─god@ubuntu  ~
╰─$
```

