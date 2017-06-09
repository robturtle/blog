---
title: About Shell
date: 2016-07-05 05:31:00
tags:
- shell
---

In this post I'm gonna discuss about the underlying mechanism that runs a shell. It's not a shell command manual. If that's what you need, please refer to [http://linuxcommand.org/learning_the_shell.php](http://linuxcommand.org/learning_the_shell.php).

<!-- more -->

## Machine & Primitive Language

All computers can be modeled as a simple abstraction -- a machine taking inputs and giving corresponding outputs.

```
input -> processing -> output
```

A computer can be distinguished by its core -- the processing logic, which is determined by the electron circuits. To best reuse the basic fatilities, a logic substitution feature is desired. We call it general purpose computers, or we can see it as a domain specific computer which taking logic as input. We can now see the meaning of program -- processing logic that in data form.

By representationaling logic as data, our computers ought to "understand" it. A language that is naturally understanded by the machine, i.e. instruction set, is called the primitive language of the machine. By communicating in primitive language, we are able to make use of the full power of the machine.

## OS & C Language

Operating system are to ease the effort operating the machine, by the mean of encapsulation and abstraction. With OS users are escaped from talking with the ugly and complex hardwares but a elegant and simple computing model. One of the most important abstraction provided is its supporting of concurrent multi-tasking.

In the OS level, a in progress running program is called a process. A process is representationalized as an in-memory data structure. In the structure, the program context, environment variables, and other necessary informations are stored for restoring use.

Since all the POSIX and almost all other OS are implemented in C langugage. We can call them virtual machines whose primitive language is C language. Through their C API we're able to make use of their full power including processes.

## Shell in a nutshell

What are those all above related to the shell? Basically a shell is a primitive interface allowing input/output interaction between users and the OS.  It's so primitive that it's like a steal shell cover surrounding the internal machine. Users are not really, but almostly facing the internal machine directly.

A shell first of all is just a normal user program that served as a entry point of other programs. From this aspect a shell provides a utility to swtich processes.

Secondly it also provides a virtual machine which has its own primitive language --- shell script language, a special language where string as its primitive type.  From this aspect, a shell runs a REPL for the shell script language.

### Syntax of shell commands

The syntax of shell commands is simple: tokens are separated by whitespaces and each of the token is called an argument. Considering the following shell command:

```shell
echo hello shell
```

It consists 3 arguments namely `echo`, `hello` and `shell`. Since whitespaces are reserved as separators, assignments requries no whitespaces surrounding the equal sign for the sake of ambiguity.

```shell
a=hello
```

All arguments are considered as a string literal except for those begin with a dollar sign, which will be considered as the evaluation of the corresponding variable. Say `$a` is the evaluation of variable `a`.

### Command execution

#### Locating program

Considering the command below:

```shell
echo hello world
```

After arguments were splited, the first argument played as a clue for the execution. Normally there're two cases:

1. The first argument is an valid filesystem path, either relative path like `./a.out` or absolute path like `/usr/bin/echo`. Then the shell will chose the file related to the path as the program.
2. Otherwise, the shell will try to concatenate each directory path in the `PATH` variable with the first argument, from left to right in the definition order in `PATH` variable. Once the concatenation forms a legal path, its corresponding file will be chosen as the program.

The value of `PATH` is a simple string where each directory path are separated with ":" character. We can use `echo` command to print out the value of it:

```shell
$ echo $PATH
/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin
```

Let's assume we input "python" to the shell. After we pressed enter, it will find a file named "python" in the order as `/usr/local/bin`, `/usr/bin`, `/bin`, `/usr/sbin` and `/sbin`. We can use `which` command to check where exactly it was found.

```shell
$ which python
/usr/bin/python
```

> Sometimes a true program is overlapped by a shell alias. For instance normally command `ls` is aliased. In this case we can use `type -a` to print out all matches.

#### Process creation

Once the program was located, the shell could then create a new process for it.  As mentioned before, environment variables are copied into the new forked process. In the shell environment, environment variables are those who were exported.

```shell
a=yooo # a is just a local variable which will not be copied to any forked process
export a # now a is environment variable
```

The `env` command can be used to print out all the environment variables.

```shell
$ env
LANG=en_US.UTF-8
PWD=/Users/YangLiu
SHELL=/bin/zsh
PATH=/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin:/Library/TeX/texbin:/Users/YangLiu/bin:/Library/Frameworks/Python.framework/Versions/Current/bin:/Users/YangLiu/.cabal/bin:/Users/YangLiu/.meteor
HOME=/Users/YangLiu
CLASSPATH=/Users/YangLiu/include/java//javarepl.jar:
a=yooo
```

Arguments (including the first one) are then passed to that program. Let's look at the main function signature in C language:

```c
int main(int argc, const char * const argv[]);
```

The `argv` is the passed argument list. For example:

```shell
 $ python --version
```

The shell first locates the program in `/usr/bin/python` (The result differs between systems), loads its code and passes the arguments. For the C program "python", the accepted arguments are:

```shell
argc == 2  # argument count
argv[0] == "python"
argv[1] == "-v"
```

For a shell script, arguments are refered by numeric variable like `$0`, `$1`, etc.

```shell
# file: lalala
echo $0
echo $1
```

A demo execution looks like this:

```shell
$ ./lalala hahaha
./lalala
hahaha
```

#### Practice

Reconsidering the forking mechanism of process system, please investigate where the information of working directory is stored in a process's data structure. And try to understand why the following shell script doesn't work as expected.

```shell
# file: wrong.sh
cd /
```

We will not see the working directory changed in the calling shell after the execution. Why?

#### Input & return value

Considering the previous model where we split computation into 3 steps, namely "input", "processing" and "output". A problem will be how does the program knows when the input was complete?  Normally the cases are:

1. The program need no input, hence no problem at all;
2. It choses a special character as the terminator, for instance, command `read` choses return character as terminator;
3. 
3. 进程读取完全部输入。这样我们就需要一个标记输入已结束的状态。我们使用 `EOF` (End of File) 来表示，在 C 里常用 -1 表示。即相当于输入在读取到 `EOF` 这一虚拟字符时停止读取输入。（但其实`EOF` 不是字符，这就导致了 C 新手可能会碰到的用 char 类型接受 `getc()` 函数的陷阱，详见《C traps & pitfalls》）。
4. 当进程被操作系统的信号中断时，默认退出。（既然是默认即说明，可设置忽略某些信号，可设置适当的中断处理函数避免退出）。

这里说的返回，就是 c 里 main 函数里的 return 语句。其中 0 代表一切正常，反之代表出现了错误。我们可以用 `$?` 这一特殊变量查询上一个执行进程的返回值。

```shell
$ ls
...

$ echo $?
0

$ ls nonsense
ls: nonsense: No such file or directory

$ echo $?
1
```

### readline 按键约定

当输入是文件时，我们可以很容易判断输入何时结束。可是当输入时标准输入（即键盘）时，我们没法用直接的方法告诉系统输入何时结束。所以我们有了一系列的按键约定：

1. `Ctrl-D` 代表 `EOF` 。我们一般喜欢用 `^D` 这样的简写来表示这是一个 `Ctrl`+`D` 的按键。要注意一点：Unix 中要求所有文件必须以空行结尾。对于标准输入这一虚拟文件来说，它同样遵守这一约定。因此：`^D` 仅当你在一个空行时才有效。
2. `^C` 代表 `SIGINT` 这一中断信号。用来强制退出某进程。
3. `^Z` 代表 `SIGTSTP` 信号。用来挂起进程。被挂起的进程可以使用 `bg` 命令在后台继续运行。

除此以外还有一系列模仿 Emacs 按键绑定的按键系列。这些被制作成一个叫做 `readline` 的库供大部分 shell 使用。

考虑命令行，它作为一个普通程序，它的结束模式是怎样的呢？显然是读取到输入结束为止。因此，我们在标准输入里输入`^D` 即等价于退出该命令行。一般来说大部分的 REPL 都遵循这一约定（`python` 程序以前需要 readline 库的支持，不知道现在是否已经内置了）。因此，我们可以通过连续按`^D` 从多个嵌套 REPL 里退出回来：

```shell
$ sh
sh-3.2$ bash
bash-3.2$ python
>>> ^D
bash-3.2$ ^D
sh-3.2$ ^D
$ ^D
[ 退出最外层 shell 了 ]
```

对于其他读取到输入结尾的程序来说，我们也需要用 `^D` 来指示结束。比如我们用 `cat` 来写入一个文件：

```shell
$ cat > file
lalala
hahaha
yooo
^D

$ cat file
lalala
hahaha
yooo
```

## 其他琐碎

### 参数分类

前面我们说了，命令行的语法实质就是空格分隔的参数列表，这在 main 函数的签名里也可以看出来，所有的参数都是平等的。不过我们也可以根据参数的形式和位置来进一步分类。对一个典型的命令行的命令，我们可以初略地把语法写成这样：

```shell
command -options arguments -options -- arguments
```

1. 因为第一个参数被用来寻找需要执行的程序，我们称这个参数为 `command`，所以我们常用这第一个参数指代命令或程序本身；
2. 我们把 '-' 打头的参数称为 `option` 即选项，他们一般是用来调整程序的行为的；
3. 其他非 '-' 打头的参数我们继续称为 `argument` 即参数，我们一般用 `argument` 来指示程序的输入。（但也未必，这完全取决于作者的喜好。比如 `git clone` 里这个 `clone` 就不是表示输入。我们一般把这种比较”后现代“风格的参数起个名字叫 `subcommand` 即子命令。从这个例子也可以看出我们完全可以根据需要构建自己需要的抽象分类）
4. 一般来说我们约定中有个特殊 `option` 是 `--`，在它之后的所有参数都不被看作选项。这是为了避免某些脑残把一些文件起了个 '-' 打头的名字之类的。

现在我们看看一个具体的例子：

```shell
gcc -std=c++11 libmylib.a -I ~/include -- --brain_fucked_filename.c
```

对各个参数分类的工作留作练习。另外要注意，有些参数是从属于前面的选项的而不是属于命令的。在这里就是 `~/include` 从属于 `-I` 这个选项。

### Shebang

前面我们说了，如果文件不是二进制，将由 shell 本身来执行文件里的内容。可有的时候，我们希望用其他的解释器来执行。所以 Unix-like 的 shell 里一般提供一个功能，我们可以用一种特殊的称为 shebang 的首行注释来提示 shell 应该载入哪个其他的程序代码。比如我们可以：

```python
#!/usr/bin/python
import sys
print "hello, python"
```

shell 就会先载入 python 的代码，再执行剩下的内容。不过有一个问题，就是在其他人的机器里，python 未必是安装在这个地方，而 shebang 又不支持 PATH lookup，只能写文件的绝对路径。那么怎么模拟 PATH 查找的过程呢？我们可以用之前提过的 `env` 命令：

```shell
$ /usr/bin/env ls
```

等价于：

```shell
$ ls
```

所以我们的文件就可以写成这样：

```python
#!/usr/bin/env python
import sys
print "hello, python"
```

现在我们看一个好玩的例子：

```shell
#!/usr/bin/env ls
# file: blahblah
```

```shell
$ ./blahblah
```

考虑一下，输出是什么？

```shell
$ ./blahblah .
```

这样呢？

如果不能理解为何会输出这样的结果，不妨把上面的命令执行流程再看一遍。

### redirect & pipe

此处关于 shell 如何控制输入的来源和输出的目标地，比较简单，自行查阅资料即可。

### background execution

我们可以通过把 `&` 加在命令行后面让程序在后台运行。不过嘛。。。你如果想让你的程序作为守护进程(daemon) 而不被挂起或杀死的话，这样做是不保险的。具体来说，除非系统 `huponexit` 为 `off`，或者用户退出前和该进程解绑，该进程才不会被退出时的 `SIGHUP` 信号终止。而解绑的进程最好还不能和标准 IO 有交互。嘛，研究得那么麻烦，还不如去好好学一学`screen` 或 `tmux` 呢。

另外对许多 web 框架，它们也提供一些运行守护进程的方法。比如`node` 的 `forever`。

#### 小结
以上便是 shell 的命令执行模型，总结一下：
1. 当命令行输入之后，如果它是一个 shell script 语句（比如 `a=hello` 赋值语言，或 `if` 语句等）则由它本身执行该效果。所有 `which` 报告为 built-in 的命令也将由 shell 自身执行效果，比如 `cd` 和 `echo`。
2. 否则，将命令行按空格分割成参数列表，将第一个参数作为线索寻找对应文件。如果它是一个路径则直接找到，否则，在 `PATH` 变量里依次寻找该文件。
3. 当找到文件，创建一个本 shell 的镜像进程。如果是二进制，则讲该文件代码覆盖原始代码，否则不覆盖。
4. 接着将参数列表传递给新的程序，并开始执行。
5. 当程序执行完毕后，控制权将重新交回原始 shell 。

