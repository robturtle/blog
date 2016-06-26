---
title: 用 Java 写一个玩具函数式语言之一
date: 2016-06-24 16:15:27
tags:
- FP
- PL
---
> 这段系统趁着放假打算稍微系统地扫一遍 PL 相关的知识。偶然间看到了王垠那篇号称价值 5 美金的文章《如何写一个解释器》，顿时便有了自己也写一个的冲动。我认为我这个安排是极好的，虽然目前脑子里的都是些零散的知识，开发时没法借鉴前人经验，但这样在编写时，所有的难题也都是靠着自己思考解决的。有了这么个实践经验后，再去看 PL 以及编译相关的书，估计能够理解的更加深刻。


自己实现一门语言一定是增加自身对编程语言理解的一种有效方法，你只有真正地尝试了一遍之后，才知道一门语言的哪些特性是精妙的，哪些是无谓多余的，哪些又是属于作者偷懒，疏忽或者出于历史原因的妥协，从而收获更好的鉴赏力和品位。

王垠的版本是用 Scheme 来写 Sheme 解释器，这个用来做传达解释器的知识脉络是绰绰有余了，不过在实现上因为借用了相同的数据结构和系统设施，就显得有些取巧了。所以我考虑用 Java 来写一个玩具 FP 语言，以便更好地体会内部细节。

<!-- more -->

因为 Jaskell 是一个确实存在的 Haskell JVM port，我这个语言就没法取这个名字。为了不要让第一次写的垃圾占据掉好的名字，我决定给它起名叫做 Toyaskell， 听上去略有点东瀛风。。设计这个语言的目的主要是以传递概念为主，所以不做任何优化。此时正值我计划把现有代码全部重构进入第二次迭代，所以才动了写这篇文章的想法，来把目前写好的一些片段记录下来。（是的它们连版本库都没有机会进入）

目前已经写好的文件架构大致如下：

```shell
.
└── toyaskell
|   ├── bootstrap
│   ├── Parser.java
│   ├── REPL.java
│   └── Tokenizer.java
├── system
│   ├── Context.java
│   ├── InterpretException.java
│   ├── LookupError.java
│   ├── PatternMatcher.java
│   └── lib
│       ├── Assign.java
│       ├── Cons.java
│       ├── Negate.java
│       ├── Quote.java
│       ├── Succ.java
│       ├── SystemContext.java
│       ├── Trunk.java
│       └── Unary.java
└── type
├── Function.java
├── Lambda.java
├── Value.java
├── list
│   ├── Cons.java
│   ├── Lists.java
│   ├── Nil.java
│   ├── Quote.java
│   └── SExp.java
├── primitive
│   ├── Int.java
│   └── TrueType.java
└── token
├── Number.java
├── Op.java
├── Symbol.java
├── Token.java
└── Tokens.java

8 directories, 30 files
```

下面开始选择性地讲解：

## 类型

我们已经知道了，在 FP 中唯一的数据类型就是函数，所以我们可以让我们的语言全部都属于 Lambda 类型，在 Java 下面这很容易定义：

```java
interface Lambda { Lambada _(Lambda arg); }
```

lambda 是一个接受 lambda 并返回 lambda 的一元函数，嗯这很函数式。

那么如何表示值呢？上次说了，值就是个常函数，所以我们可以这样：

```java
abstract class Value implements Lambda {
    Lambda _(Lambda arg) { return this; }
}
```

好了有这就是我们所有需要的类型结构了。不过为了让我们的语言能够真正操作一些实体，我们还需要定义一些具体的类型。现在作为演示用我们暂时只定义一个基本类型，那就是整数类型：

```java
class Int extends Value { public final int val; }
```

由于整数是个两端无穷的可数 (countable) 集合，我们只需要定义几个原子操作，其他的在整数上的操作都可以由他们推导出来。理论上说，如果我们能够选定一个正数基点和一个负数基点，我们只需要 + 这一个操作符便可遍历整个整数数列。不过出于对传统的 FP 的模仿，我们可以定义另几个等价的操作：

```java
Lambda succ = (Int) x -> new Int(((Int)x).val + 1);
Lambda neg = (Int) x -> new Int(-((Int)x).val);
Lambda IF = x -> a -> b -> ((Int)x).val == 0 ? b : a;
Lambda GT = a -> b -> t -> f -> ((Int)a).val > ((Int)b).val ? t : f;
```

(注意这段骚气的代码只能在 Java 8 下编译)

有了这几个操作我们便可以推导出所有在整数域下的运算了，比如我们可以定义递减和加法：

```java
Lambda pred = (Int) x -> neg._(succ._(neg._(x)));
Lambda add = (Int) a -> b -> IF
  ._(a)
  ._(add._(pred._(a))._(succ._(b)))
  ._(b)
```

当然这里这是为了展示可行性而已，实际实现自然可以利用现成的系统功能来做。因为从数学上角度讲，我们只需要少数几个运算符和单一的类型便可以完成所有操作，所以我们也可以装逼地讲其他的包括类型系统和各种多余内置函数通通都是语法糖 : ) 。

## 拓扑

好了现在我们已经有了整数集合，我们可以用它来表示所有离散的事物，当然也包括字符。不过我们发现我们目前为止还只能表示单个的点，还没有办法表示结构，意即点与点之间的关系。为了表示点与点之间的关系，我们只需要表示两点之间的关系就够了。所以我们再添加一种类型，我们叫它链表：

```java
abstract class SExp extends Value { abstract boolean isNil(); }

class Cons extends SExp {
    public Cons(Lambda head, SExp tail);
    boolean isNil(){return false;}
}

class Nil extends SExp {
    boolean isNil(){return true;}
}
```

这里借用了很多 FP 用来表达链表的用词，他们把 nil 看作空链表（同时也用来表达 false)，用 cons 即 (construct) 来表示一条边。有了它，我们可以表示所有的数据结构，包括各种树和图。这也是为什么当年真的有 Lisp 程序员会声称其他的数据结构都是冗余，我们只需要链表就够了！

另外我们发现，当我们解析代码时，我们会先构造出一个抽象语法树（AST），既然是树那我们就同样可以用链表来表示咯。而且我们的语法也可以设计的方便 AST 的构建。于是乎，代码的结构和数据的结构我们都只需要用这一种结构来表示咯，我们把这种数据结构称为 S-表达式。

不过当我们同一种结构既表示代码也表示数据的时候，就容易产生歧义，解释器就不知道这里究竟是看成数据还是看成代码。为了便于区分，我们得给表示数据的链表做个标记。我们用 quote 表示。

```java
class Quote extends SExp { public final Cons list; }
```

## 分词

好了类型都准备好了，我们可以试着实现解释器的第一步，就是把一个无结构的字符串，解析成一个由 token 组成的 AST 树。 token 表示的就是最小的语义单元，比如一个变量名。分词这件事情还是很简单的，有人说它简单到甚至不能被算进编译技术领域之内。不过需要注意的是，除非是特别简单的正则文法，我们尽量不要用正则表达式来做分词。一是因为它只能处理上下文无关的正则语法的局限性，二则是因为它实在不够灵活。下面我们简单地写一个 lisp like 的语法的分词器，它的语法大致像是这样：

```shell
(print (+ 1 2) 3 4)
```

其中一个括号就代表一个子链表，链表可以无限嵌套。我写的分词器的核心代码类似这样：

```java
private static void processChar(int c) {
    if (c == '(' || c == ')' || Character.isSpaceChar(c)) {
        pushToken(tokenStack);
    } else {
        wordCache.append((char) c);
    }

    if (c == '(') {
        tokenStack.push(new ArrayDeque<>());
    } else if (c == ')') {
        Deque<Lambda> tokens = tokenStack.pop();
        if (tokenStack.isEmpty()) {
            throw new InterpretException("ERROR: additional right bracket");
        }
        SExp list = Lists.build(tokens);
        tokenStack.peek().push((Lambda) list);
    }
}
```

可以看到逻辑还是很清晰的，而且其实我们可以看到其中蕴含着几层抽象。比如 '(' 和 ')' 实际上就是上下文切换的开关，如果有需要，我们可以通过切换上下文来实现作用域的概念。另外一个抽象就是分隔符，在这里就是 '('，')' 和空白字符。在这里出于简洁需要我把他们都写死了，但实际上我们可以把他们抽象成一个
（上下文，分隔符）二元组并提供它们的切换语义。有了这层抽象我们甚至可以允许用户自定义自己的上下文切换！（实际上在大部分基于原型的语言中这很常见，人们也乐得利用基于原型的语言的强大灵活的语法来制作各种领域相关语言(DSL)）。通过提供一组上下文切换符和对应的分隔符已经切换的动作，我们实际上就把这个有限状态机给完全声明出来了。编程语言里常见的上下文切换有大中小括号（常用来表示作用域），'/' (常用来表示正则表达式字面量)， 以及用来表示字符串的引号。

从这里可以看出，要实现 block scoping 的功能，简直不能再直白了。但 JavaScript 的作者却偏偏不实现，我认为这绝对是一个错误。

## 闭包

在实现解释器之前，还得先考虑如何实现闭包。像之前讲的那样，如果我们只部分地绑定了参数，那么它返回的就是一个接受剩余参数的函数了。当绑定发生时，返回的函数必须想法把当前的上下文以及这个绑定都存储在某个地方。可选的方案很多，比如我们可以定义一个闭包类，绑定环境与函数，像这样：

```java
class Closure {
    public final Lambda func;
    public final Context context;
}
```

但是如果是这样的话，我们的 Lambda 就没法获得必要的参数信息，故而我们需要显式地实现我们真正运算的部分，比如这样：

```java
class Add extends Computation {
    Value compute(Context context) {
        Int a = (Int) context.lookup("a");
        Int b = (Int) context.lookup("b");
        return new Int(a.val + b.val);
    }
}
```

当然，我们还有另一种方案，那就是让 Lambda 自己来管理上下文，这样就不需要额外的 Closure 类和 Computation 类了，还有一个额外好处就是，这样 S-表达式也是自带上下文的了，这样实现 scoping 简直不能更简单了。不过这样 Lambda 的代码就会相对复杂些。

```java
public class Lambda {
    protected final Symbol formalArg;
    public final Lambda returnValue;
    protected final Context closure;

    public Lambda(Symbol formalArg, Lambda returnValue, Context outer) {
        this.formalArg = formalArg;
        this.closure = new Context(outer);
        this.returnValue = returnValue;
    }

    public Lambda call(Lambda actualArg) {
        return returnValue.apply(formalArg, actualArg, closure);
    }

    protected Lambda apply(Symbol formalArg, Lambda actualArg, Context outer) {
        Context context = closure.cloneWithParent(outer);
        context.bind(formalArg, actualArg);
        return new Lambda(this.formalArg, returnValue, context);
    }
}
```

我目前选择了第二种方案，不过目前发现这样的话代码耦合度略高且不易扩展，所以之后估计会改成第一种的形式。当然，随着正式学习 PL ，这些方案可能都会受到影响。

## 解释

好了终于到了写解释器的时候了。第二种方案妙的地方在于，Lambda 已经把逻辑处理的大头都搞定了，解释器需要做的就非常少了。少到可以全文贴过来。

```java
public class Parser extends Lambda {

    public Parser(Context outer) {
        super(null, null, outer);
    }

    @Override public Lambda call(Lambda lambda) {
        if (!(lambda instanceof SExp) || ((SExp) lambda).isNil()) {
            return lambda;
        }

        Cons list = (Cons) lambda;

        Lambda value = null;
        for (SExp exp = list; !exp.isNil(); exp = ((Cons) exp).tail) {
            Lambda current = ((Cons) exp).head;
            if (current instanceof Symbol || current instanceof Op))) {
                current = closure.lookup(current);
            }

            if (current instanceof Function) {
                current = ((Function) current).cloneWithContext(closure);
            } else if (current instanceof Number) {
                current = ((Number) current).value;
            }

            if (value == null) {
                value = current;
            } else {
                value = value.call(current);
                if (value instanceof Function) {
                    value = ((Function) value).cloneWithContext(closure);
                }
            }
        }
        if (value instanceof Cons) { value = call(value); }
        return value;
    }
}
```

总体上说这个解释器做的就是从左到右依次两两合并 lambda 并最终返回一个唯一的 lambda 的reduce 的过程。从这里也可以看出为什么科学家那么爱 lambda。就是因为它的简洁性和体系的自洽性，使得对它的处理也变得简单无比。解释器需要额外处理的，仅就是给函数注射入上下文而已了。

## 惰性求值

除了闭包，还有一个重要的特性我们可以实现的就是惰性求值。从上面的代码我们可以看出，当 Lambda 进行参数代换的时候，它做的就真的只是简单的替换，即使它是一个可运行的 S-表达式。那么这个表达式什么时候才运行呢？我们可以放到它真正进行运算的地方，也就是末端的函数：

```java
@Override Lambda apply(Symbol formalArg, Lambda actualArg, Context outer) {
    // this is where lazy evaluation happened
    if (actualArg instanceof Cons) {
        actualArg = new Parser(outer).call(actualArg);
    }
    return actualArg;
}
```

## 系统库

现在万事都具备了，只差把一些必要的基础运算实现并且注入到默认的上下文当中了。我们目前定义了几个默认绑定：

```java
import static toyaskell.type.token.Tokens.build;

public class SystemContext {
    public static final Context systemContext = new Context();
    static {
        systemContext.bind(build("t"), trueType);
        systemContext.bind(build("nil"), nil);
        systemContext.bind(build("++"), succ);
        systemContext.bind(build("neg"), negate);
        systemContext.bind(build("cons"), cons);
        systemContext.bind(build("let"), assign);
        systemContext.bind(build("'"), quote);
    }
}
```

## REPL

然后我们就可以写一个不断读取输入不断返回计算结果的程序了，我们称之为交互式编程环境，或者 REPL。

```java
public class REPL {
    public static void main(String[] args) {
        Parser parser = new Parser(systemContext);
        Scanner stdin = new Scanner(System.in);

        do {

            System.out.print("> ");
            System.out.flush();
            String input = stdin.nextLine();
            if (input.trim().equals("quit")) { break; }
            try {
                SExp exp = tokenize(input);
                System.out.println(parser.call(exp));
            } catch (InterpretException e) {
                System.out.println(e.getMessage());
            } catch (ClassCastException e) {
                e.printStackTrace();
            }

        } while (true);
    }
}
```

就这样一个简单的解释器就完成了，我们试一下：

```shell
$ java -cp . toyaskell.bootstrap.REPL
> ++
x -> x
> ++ 1
2
> ++ (++ 1)
3
> cons
x -> xs -> xs
> cons 1
xs -> xs
> cons 1 (cons 2 nil)
( 1 ( 2 nil ) )
> let x (cons 1 nil)
nil
> x
( 1 nil )
> let y x
nil
> y
( 1 nil )
```

从这次实现我理解了为何赋值语句往往返回空值，这很可能是出于惰性求值的考虑，因为如果要打印这个值，那么就必须求解这个值，这在强制要求惰性求值或者面对一次性的数据是不适用的，比如一个 iterator 或者 generator。

另外一个最重大的缺陷就是，目前这个语言是没有类型系统的，因为所有的数据都被看作 lambda 了，或者说被看成 S-表达式了。当然好在我们可以利用 Java 的类型系统来为我们做错误提示：

```shell
> ++ y
java.lang.ClassCastException: toyaskell.type.list.Quote cannot be cast to toyaskell.type.primitive.Int
    at toyaskell.system.lib.Succ.apply(Succ.java:15)
    at toyaskell.type.Lambda.call(Lambda.java:19)
    at toyaskell.bootstrap.Parser.call(Parser.java:45)
    at toyaskell.bootstrap.REPL.main(REPL.java:24)
```

但我想不到任何一个理由让一个语言的作者放弃类型系统，像之前很多所谓的弱类型系统和动态类型系统，我严重怀疑这其实是作者懒惰的结果。另外对于 lisp 这门语言本身，现在的感觉就是它简直只能算作是半成品。比如所有操作符都用前缀顺序表示，就只是为了方便解析而已。然后人为了照顾解释器，转而去适应手写语法树的风格。它里面很多的奇技淫巧，比如宏，讲真我觉得就和我上面用 pred 和 succ 实现加法运算差不多。语言本身提供的糖太少，提供的抽象太少，而且提供的约束还少（类型系统），这只会导致一个门槛高且低维护性的语言。

罢了，下一个版本估计会在正式看了 PL 和编译原理一段时间后了。这期间可能会重用写好的 tokenizer 来做一个基于 Annotation 的声明式测例生成库。这堆代码就暂观后效吧。
