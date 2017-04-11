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

打开 virtualbox 自身的设置选项，在 Network 标签页上可以看到我们已经有了一个默认设置好的宿主网络配置，如果没有，就自己新建一个就好了。

![](https://www.dropbox.com/s/xp0jfcvv757b8qf/Screenshot%202016-10-17%2021.11.20.png?raw=1)

### 2. 虚拟机建立 Host-only Network Adapter

打开虚拟机的设置面板，在 Network 标签下，新建一个 Adapter，选择为 Host-only Adapter 即可。

![](https://www.dropbox.com/s/xp0jfcvv757b8qf/Screenshot%202016-10-17%2021.11.20.png?raw=1)

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

![](https://www.dropbox.com/s/fbv8btuvo02ejae/Screenshot%202016-10-17%2021.14.03.png?raw=1)

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

## 便利设施

接下来讨论怎么配置一个舒服的操作环境。

### 起名

当 ssh 的目标多了之后还用 IP 来访问的话就显得有些麻烦了。简单，在 `/etc/hosts` 上加个别名就好了。虚拟机就按操作系统来命名比如 `ubuntu` 这样，VPS 就用提供商来命名比如 `linode` 。

### ssh 免密码登录

在 `/etc/sshd_config` 的默认配置里，root 是禁止了密码登录的方式的。如果是外部机器，我们最好还是保持这个设定不变，用一个普通用户来设置 root 的信任。不过既然我们在本地虚拟机里，直接把 `PermitRootLogin` 改成 `yes` 就好了。

接着就可以用 `ssh-copy-id` 来给远程端添加信任了。这里要注意一下多证书存在的情况，跑之前最好用 `ssh-copy-id -n` 确认一下。和它手册上写的不一样，当我没指定证书的时候，程序并没有使用 `id_rsa.pub` ；不过当我用 `ssh-add` 把默认证书添加给 `ssh-agent` 之后，`ssh-copy-id` 倒也能正确使用默认的。感觉这个是 machine depended 的，之后把这个步骤做成部署脚本的时候还是要记得显示指定证书。

> 事实上我现在越来越倾向默认值是一个坏特性，想想 null 和 undefined，另外就是它的默认取值可能是平台相关的，想想 C 里 int 实现的字长。这个在之前[讲代码兼容性的文章](http://blog.yangliu.online/2016/03/17/some-cpp-compatibility-issues/)里提到过。部署系统的一大核心诉求就是消弭平台间的差异，这其中最主要的工作就是对默认值的对抗上。

### Emacs 远程访问

配置 Emacs 的远程访问模块 Tramp 的时候确实发现个比较难解的 bug，表现是当访问到远程地址的时候就立刻假死了。在排除了是 helm 的问题之后，在网上找到了解答。是 Tramp 对 zsh 的一些花哨设置表示出了不解（这个 oh-my-zsh 还和某个 Ruby 版本管理在某个发行版上有兼容性冲突，我现在也在非常认真地考虑是否要转回 bash）。好在这次用了网上的解答，倒是简单地加了[几行兼容性处理](https://github.com/robturtle/.zsh/commit/c638cecaf66ace805cf4afb322e8277fb0b6db0a)就搞定了。

配置 Emacs 的嵌入终端 multi-term 的时候出现的 bug 就更隐晦了。总之任何有关 shell 的配置的事情，都涉及了无数的历史遗留、谜题和陷阱。而且因为 multi-term，zsh，以及 Emacs 本身都是非常脆弱而显得非常 bug-prone 的，一旦出现 bug 往往需要花上好一段时间才能判断问题究竟出在谁身上。在我目前的 Emacs 里，已知现存的终端相关的 bug 就还有不下 3 个。不过在非常小心地解决掉影响日常使用的之后，用起来还是算方便的。

综合以上的便利设施，便可以好好享受 Emacs 下远程端虚拟端各种文件系统无缝切换的爽快了。
