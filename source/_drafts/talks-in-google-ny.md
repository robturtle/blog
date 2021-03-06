---
title: 造访谷歌纽约
date: 2016-12-22 19:48:12
category: chit-chat
tags:
---

> Kurt 先生在 2010 年从 Poly 博士毕业后进入谷歌，现已在谷歌工作了 6 年，期间曾任 Chubby 系统的 leader 。他根据其在谷歌开发分布式系统的经历，回母校开设了分布式系统的课程。课程结束后，非常有幸受他邀请，造访了谷歌在纽约的总部，和他讨论一些本人感兴趣的话题。

## 闲聊

我：平时的作息起居是怎样的？

K：一般中午12点到公司，工作到晚上 9 点回家。

我：话说谷歌的饭堂果然不怎么样呀……

<!-- more -->

![](https://www.dropbox.com/s/szse5w79wpsk6xs/google-cafe.jpeg?raw=1)

K：是的呢。不像 LinkedIn ，他们虽然没什么优秀的工程师，但他们的饭真的好好吃。

我：那你平时吃啥呢？

K：Chicken over rice.

我：为什么选择待在纽约办公室？

K：Everything can happened in 6th Street.

## 关于工作

我：现在在干什么项目？

K：配置系统。

我：像 ansible, puppet 那样的？

K：对。我有个 LinkedIn 跳过来的同事，他说他们就用的 puppet，不过我们会希望做一个更加并发，依赖更加层次化的系统。

我：怎么会有这么一个项目？

K：我自己的主意，我觉得这个地方可以改进一下。同时这样搞点事情出来，可能也更容易升职。

我：现在进展如何？

K：我之前花了整整半年来熟悉旧系统的代码。现在我一行代码也不写，主要的工作就是到处开会，说服那些比我高级的工程师投入到这个项目里来。

我：你花整整半年时间只读代码不承担任何其他任务，公司那边你是怎么交代的？

K：一方面那半年我还承担 Chubby 系统的顾问工作，另一方面我和上司进行了充分的沟通，让他理解这是一个预计两年才能完成的任务。

我：两年这个时间是怎么来的？

K：我自己根据任务量估计的。总体来说在这里你还是有充分的自由去干你认为重要或者有意义的工作的。

我：你的职位是 Server Reliability Engineer，能介绍一下这个职位和开发有什么区别吗？

K：SRE 团队和开发团队有很大的重叠，（包括我在内）估计只有四五个人是完全负责 SRE 而不管开发的，而且人员也经常更换各自的职责。唯一的区别估计就是 SRE 有 on call，当然 oncall 的工资也高点。

我：on call 的时候大概真正被召唤的频率是怎样的？

K：大概一天一次吧。（注：根据在亚马逊实习的小伙伴透露，这个频率相当低了。不过我个人并不确定是否具可比性。）

我：你觉得你未来还会重新开始写代码吗？

K：Definitely.

## 关于编程

我：所以，你们写 C++ 真的完全遵守 Google Code Style？真的完全不用异常？

K：那是必须的。异常这个反正我在 Chubby 里是不用的。至于规定嘛，我查查……嘿！还真的完全禁止用呢。

我：你的开发工具是怎样的？

K：vi

我：你不觉得 vi 基于正则的高亮和缩进很蛋疼吗？

K：I just get used to it.

我：那你怎么调试呀？

K：我几乎不用调试器，我平时最常用的是…… printf

K：这么多年了，我也就大概只有几次感觉到我的编辑器令我痛苦，就是当打开少数非常长的源代码的时候，以及打开有非常多非常相似签名的重载函数的项目的时候。不过这个时候闪现在我脑海里的，不是去换个编辑器，而是把这些代码重构了。
