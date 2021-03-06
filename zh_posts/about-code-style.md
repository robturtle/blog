---
title: 关于代码规范
date: 2016-03-28 17:02:32
categories:
- chit-chat
tags:
- Java
- "code style"
---

> 最近小组林生提议看看九章算法的视频，第一节课讨论代码规范（干得漂亮！），顺便记录一下编程以来碰到的印象最深的几个规范问题。

所有的新人都难以意识到代码规范的重要性，除非他天生是处女座。然而这件事情却是无论如何去强调都不为过的 --- 对代码的格式一致性的极致追求，是体现一个代码编写者责任感的最基本素养。虽然最近十年它已经获得了相当程度的重视，不过时至今日新人仍然难以从语言入门教材中完全掌握代码规范。因此，工匠时代的“传带帮”教学模式依然是帮助新人迅速提高的不二法门。

视频15分钟，讲师拿出了一个缺乏规范的代码，指出这种代码在面试中最吃亏的地方就在于暴露了自身编程年龄过短的事实，然后他就不再继续解释下去了。这也是另外一个指导入门者的一个常见误区，只给出后果，不给出理由。于是我们可以看到很多新手墨守自己学的那套所谓规范，并且以为代码规范只是为了遵循审美上“好看”的一种强迫症要求。这显然是非常片面的。

引用 Paul Graham 的话说，“程序是提供给人类阅读的，只是它恰好能被机器执行而已”。当前的软件行业现状正是如此，降低合作门槛加强沟通效率是比想办法降低你的程序的能耗更节省成本的办法。而代码规范就是在多年合作实践中被广泛接受的，提供给人眼这个分析器的一系列编码标准。为了达到这一目的，代码规范主要考虑两个方面，“可读性”和“普适性”。其中可读性这点主要是基于人眼分辨能力做的优化，比如在变量和操作符间加空格，在不同逻辑代码段间加空行等方式加速人眼识别目标。而普适性则是考虑源码层面（有时还包括二进制文件层面）的平台一致性，典型的规定就是和字符编码，可用字符集，以及可用语法特性方面的规定。可以说，人们对代码“好看”的主观观点，其实只是大脑对于“这段代码解析起来非常方便”这一印象的反映。

接下来我想从我写代码过程中碰到并思考过的例子，给出我对此类规定的解释。考虑到是讨论规范问题，选择 Java 这一 rigorous 的语言来做例子应当是最合适不过的了。我将采用 [Google Java Style](https://google.github.io/styleguide/javaguide.html) 作为样本来举例。同时我也强烈建议所有的初学者，仔细地浏览一编此代码规范并尽量遵行。
<!--more-->

## 缩进
> 2.3.1 Whitespace characters: the **ASCII horizontal space (0x20)** is the only whitespace characters that anywhere in a source file. This implies that:
>
> Tab characters are **not** used for indentation.

这个其实是关于普适性的一个要求，如果一个程序员会忽略这件事，要么是他很少把它的代码在不同的机器，或者本地和在线阅读页面之间传来传去；要么就是他很少去修改别人的代码。而往往编程新人这两个情况同时具备。不过一旦你至少经历了一次，要么你的代码在传到网上去之后发现格式乱得一塌糊涂，要么在修改别人的代码的时候被各种缩进混杂的情况搞得焦头烂额，那么你几乎必然就会得出一个结论：tab is evil!

问题很简单，你永远无法预设 tab 的宽度在所有你使用的平台是一致的，并且由于它的不可见，你对某些情况错误混杂了 tab 和空格的状况也难以察觉，这样一旦你的代码在一个 tab 设置和你的预设不一致的地方，你的代码就会变得简直没法看了。

一般将到这个时候，初学者就会搬出一个 naive 的问题了，“那你不用 tab 改成按好几下空格，不累吗？” 所以说“too young too simple”啊，但凡他们能接触点 vim 一类的合格编辑器或者 JB 家的 IDE，你就至少对以下特性有所耳闻：

1. 编辑器大部分情况下提供自动缩进；
2. 在需要手动缩进的情况下，你完全可以用 tab 缩进，而编辑器自动将 tab 解释为空格；
3. 编辑器是可以懂得自动猜测当前代码的缩进设置的；
4. 退缩进的时候，多个空格也可以被解释成 tab。

任何一个合格的编辑器和 IDE 都应该提供这样的功能，而且因为其自动化程度，基本上是一次配置，终生无忧的。不妨尝试搜搜你最常用的 IDE，看看它的最佳 tab 配置是怎样的。

> (这里是私货:) 现在我个人比较推崇2空格缩进，这个可以说是前端行业高速扩张之后向其他软件领域渗透过去的。这种方案使得代码相对紧凑，同时在现代编辑器提供的 ruler 的帮助下，让它原来的缺点如长 block 的对齐问题变得不再显著了。

## 空格分隔 token
> 先对不熟悉的读者解释下，token 属于编译理论的术语，简单来说可以看成是源码里最小的语义单元，比如一个变量，一个操作符（如 + )

首先，用空格来分隔 token 显然有助于人眼解析，比如考虑这段 Ruby 代码，从这样：

```ruby
x=Pi*r**2+k/2>0?true:false
```

变成这样：

```ruby
x = Pi * r ** 2 + k / 2 > 0 ? true : false
```

好很多了，这也的确就是很多代码规范上白纸黑字写下来的规定。可这真的就是最好的方案吗？

至少我认为这不是，稍微和数学公式的形式对比一下：

$$
x = \pi \cdot r^2 + \frac{k}{2} > 0 \,? \text{ true} : \text{ false}
$$

显然又比上面的代码形式好读很多。为什么呢？这是因为代码的写法把操作优先级的信息给扁平化了，优先级次高的 `/` 操作紧紧地挨着优先级最低的 `? :` 操作，看上去就好像这一系列操作是按照出现顺序次序运算的。这使得人脑在解析这段代码时，还得主动脑补每个操作执行的次序。这时候一旦操作层次多个几层，就很容易引起混乱了。

这个时候，一般又会有几种方法辅助大脑解析，一种就是加上括号来指示优先级，变成这样：

```ruby
x = ((Pi * r ** 2) + (k / 2) > 0) ? true : false
```

我更欣赏另外一种，就是不按 token 来分隔，而是将表达式分成粒度更大的逻辑单元，变成这样：

```ruby
x = Pi*r**2 + k/2 > 0 ? true : false
```

其中 `Pi*r**2` 属于大部分人熟知的常识，完全可以看成一个单元，`k/2` 也类似。这样的话，不需要引入括号，也能达到比较好的显示效果。

为什么不引入括号很重要？因为括号这东西对人脑异常不友好，我个人的经验就是，人脑在解析括号嵌套达到3层以上的表达式的时候，理解的难度就开始显著增加了。所以在业界笼罩在 C 语言阴影下的日子里无数有志之士纷纷设计出了更少依赖括号的语言。

> （这是吐槽，可跳过）什么？你说 Lisp？我严重怀疑在 rainbow parenthesis 这一伟大发明之前，Lisp 真的有可读性可言？

> （这是吐槽，可跳过）而且感觉上，人脑的解析过程，貌似也是维护了一个操作符栈，一个括号就对应了至少一次入栈。我感觉我个人的栈长度只有3，多了就得有 swap 成本了。

但是最后一种方法的缺点也很明显，就是它也只能表示2层的结构关系，当操作层面上去的时候扁平化的趋势依旧无可避免（这时可能可以考虑和括号混用的形式）。至于说用空格数量表示优先级层次的，比如这样：

```ruby
x = Pi*r**2 + k/2 > 0  ?  true  :  false
```

说实在的，我应该从没见过一个活人会推荐这样的写法。这种时候如果你感觉表达式太复杂了，还是考虑拆分吧。

## 括号内侧
大部分老式的编程实践都要求括号的内侧是不留空格的，像这样：

```
compare(a, b);
```

不过不知从何时开始，内侧加空格的风格也流行了起来。我最早看到这种风格，也是在 java 代码里：

```
compare( a, b );
```

理由也和上一节一样，用分隔来辅助大脑解析。而且从我对这种风格的代码的阅读经历来说，我可以给它评价一个“效果拔群”！以往在阅读第一种风格代码的时候，往往要进入“精读”的状态下，函数调用表达式才能开始被解析；而在第二种风格下，我感觉往往在粗略地浏览状态下，也能方便地迅速掌握函数调用的大致框架。最直观的区别是，我在阅读第二种代码的时候，颈椎大部分时候是不会自主前移的。

不过这种风格也有一个非常大的缺点，这在我模仿这种风格的时候非常强烈地感受到了。我在写的时候就常常问自己一个问题，那就是嵌套调用的内部括号，要不要也要加空格？这个之所以是个问题，是因为在写程序的时候，3层以上的函数嵌套也是经常出现的，而这个时候如果每一个括号内部都加空格的话，我的观感就是变得异常地混乱。考虑下面这个表达式：

```
Math.sqrt(Math.pi, Math.pow(Math.abs(a - findNeg(b)), Math.pow(2, findNeg(b))))
```

如果全加内侧括号，就会变成这样：

```
Math.sqrt( Math.pi, Math.pow( Math.abs( a - findNeg( b ) ), Math.pow( 2, findNeg( b ) ) ) )
```

看上去好像也没那么糟糕, 甚至这个时候还比第一种风格要清晰些。但有一点必须要认识到，那就是随着调用层次的逐渐复杂，后者的情况会变得越来越糟糕。

那么就没有一个比较好的方案了吗？其实分析上面这个问题，我认为争论的核心就在于：括号不能有效地分隔单元。的确，在很长一段时间里的绝大部分编辑器下，这个断言是成立的。不过，后来我们有了彩虹括号，给括号加上了对比显著的语法高亮了，这个问题在很大程度上就被环境了。这也是为什么我在上面的例子里没有开语法高亮的原因。现在考虑加上语法高亮，是这样：

![](https://www.dropbox.com/s/6ao47f7d62muvo3/Screenshot%202016-03-28%2019.31.58.png?raw=1)

注意上图的括号。


## 逗号

逗号之所以有问题，是因为它作为分隔符，大部分情况下是一种前后不一致的存在。比如定义一个列表：

```ruby
foods = [
  apple,
  banana,
  orange
]
```

`orange` 后面没有逗号。这样写，虽然完全符合逗号的语义，但是却给程序员带来了不方便。考虑如果这个时候你要加多一个元素 `pear`，你就得先在 `orange` 那行末尾加一个逗号，再在下一行加上 `pear`，非常的 trivial。看上去好像也没麻烦多少，但是这其实和 C++ 里的 `new/delete` 类似，任何一个分成多步无法原子化的操作，理论上人类就一定早晚会犯错。特别是在早些年写编译型语言电脑性能还不足以支撑实时检查的时候，好不容易花了几分钟编译结果你跟我说少了个逗号编译失败？！

这应当就是大部分语言都提供在列表末尾加逗号合法的语法糖设计的理由，然而，还有很多使用逗号的地方没法适用列表的语法糖，比如连续声明变量。

所以，在对应那个年代的代码里，我们看过这样的 C 写法：

```c
int a = 1
  , b = 2;
```

又比如，C++ 构造函数列表。

所以在 MFC 的脚手架代码里，我们可以看到这样的写法：

```c++
Constructor()
: para1(1)
, para2(true)
, para3(3)
{
  /* ... */
}
```

这样当你需要增添新的要素的时候，你只需要修改一行，退一步说，至少也可以让加上逗号这件事情变得显而易见。

这种写法看上去很不错，我到现在也是这么认为的。可惜的是，不知怎的，这种写法并没有怎么流行开来，导致大部分编辑器的自动缩进功能都没法很好地处理这种风格。（上面的内侧加括号也存在一样的问题）那么出于惰性，也自然就会选择虽然没那么好但是编辑器支持好哇的风格了。不过若果有一天这种风格流行了，我也一定不觉得奇怪。

## （私货）左花括号换行？
到现在还在坚持左花括号换行风格的编写者，感觉上就是周身笼罩着一层 old school 的氛围，隐隐中感觉他接过了老人传递给他的 CS 新古时代 legacy 的精神衣钵。

宗教战争有害健康，不过我就问一个问题，Pythoner 怎么就愿意标榜 Python 的可读性？而本质上左花括号不换行的风格，和 Python 风格是一致的，既然 Python 有它的道理，不换行也自有它的道理。

他们说换行更可读？先不说正不正确，我只想说我的屏幕高度有限，实在没地方放这些白占地方的空行了。

# 小结
啊咧，本来想讲 Java Style 的，结果不知不觉不知道偏到哪里去了。不过也好，虽然远没达到身经百战，西方的代码我哪个没见过，但也算是分享了下人生的经验了（笑）。总的来说，代码规范在如何帮助人类更好理解代码方面提供了一套解决方案。在考虑风格的实际效果的同时，经常还得考虑它的流行程度和编辑器的支持程度。不过明确了规范的动机，在特殊情况下采用规范之外的写法，只要能够充足的理由支撑，有时也是可以接受的。写这篇文章的目地就是做个记录，今后如果能吸收些其他开发者的经验，慢慢把其他的风格选择相关的讨论思考也补充进来，就是最好的了。
