---
title: 用 Java 解释 Monad
date: 2016-08-05
tags:
- monad
- FP
- PL
---
虽然说网上充斥那么多的 Monad 理解都表明了看别人写的理解其实一点用也没有，不过又怎么样呢。我还是要写。

## 类型升阶 (type lifting)

其实和常说的 int 到 double 的 type lifting 也是一回事，这就是一个将原类型扩展为可以表示更多状态的过程。不过这里说的 type lifting 更加广义。
<!-- more -->

假设我们有个类型 Int, 可以表示整数，如果我们需要表示一个非法状态的话，用原始的 Int 类型就没有任何办法。于是我们可以把它提升到一个包含一个新的表示非法状态的类型。我们称它为 Object ，新的非法状态为 null。

```java
class Object { union { Int value, Null null }; }
```

我们可以用 null 来表示值“未设置”：

```java
class Optional extends Object { Object object; }
```

当我们在传输 Optional 的时候，我们可以用 null 来表示 “传输错误”：

```java
class Transfer extends Object { Optional optional; }
```

## 类型降阶

现在问题来了，我们希望实现一个函数，输入 Transfer ，如果具有有意义的 Int 值，则输出值，否则输出 null。我们会怎么写这个函数呢？
大概会像这样吧：

```java
Object fetchValue(Transfer t) {
  if (t == null) {
    return null; // represent tranfer error
  } else {
    if (t.optional == null) {
      return null; // represent not set
    } else {
      if (t.optional.object == null) {
        return null; // represent not existed
      } else {
        return t.optional.object.value;
      }
    }
  }
}
```

在我们构建 Transfer 类型的时候，进行了三次类型提升，那么当我们试图去取用它的原始值的时候，我们就必须原路降阶回去。在降阶的代码里面我们很明显可以看出来里面具有重复的逻辑，即对 null 值的判定分支。而且我们每次提升类型的时候，都是在原有值域里增加了同样的一个 null 值。可以理解成我们每次都提升了相同的类型，如果类型名为 Object 的话，那么三次提升后它的类型就相当于 `Object<Object<Object<Int>>>` 。

## bind

我们可以想办法把重复的逻辑提取出来：

```java
interface Fetcher<T> { Object fetch(T o); }
<T> Object bind(Fetcher<T> fetcher, Object o) {
  if (o == null) { return null; }
  return fetcher.fetch((T) o);
}
```

对 null 的处理逻辑被提取出来后，我们的取值的逻辑就可以不需要写分支判断了：

```java
Fetcher optionalFetcher = o -> o.optional;
Fetcher objectFetcher = o -> o.object;
Fetcher intFetcher = o -> o.value;
```

现在，我们把几个 Fetcher 连接起来就好了。

```java
Object fetchInt(Transfer t) {
  return bind(intFetcher, bind(objectFetcher, bind(optionalFetcher, t)));
}
```

如果我们把 bind 函数定义成一个写作 “>>=” 的二元操作符，我们的函数就能写成这样：

```java
Object fetchInt(Transfer t) {
  t >>= optionalFetcher >>= objectFetcher >>= intFetcher;
}
```

看上去就像是顺序对类型进行拆装，把本来嵌套的逻辑变扁平了。并且它还忽略了过程中的 null 处理逻辑。

现在，我们可以不严谨地说，这种支持 “>>=” 操作的类型，就是 Monad 了。

那么 Monad 究竟带来了什么呢？

我们说，Monad 提供了一种将高阶类型 `F<F<T>>` 重新降阶回 `F<T>` 类型的能力。这使得我们在多次同范畴下进行类型升阶的过程中，不需要关心类型的降阶过程，而只需要提供最低阶的操作逻辑，就可以处理任意阶数的类型的能力了。

在上面的例子里，bind 让 `Object<Object<T>>` 降阶回了 `Object<T>`。对于n阶类型 `Object<...<Object<T>…>` ，我们只需要将 n 个 bind 函数链式调用，就可以保证操作始终在一阶类型 Object 上。这种由自身映射到自身的类型，即是自函子了。

同样的道理，我们可以用类似的操作把 `List<List<T>>` 降阶成 `List<T>`，让 bind 内部实现去关心空列表或 null 的情况，而我们的实际逻辑代码始终可以假定列表是可以正常处理的状态。 这样，List 也被实现为 Monad 了。

当然，实现 Monad 需要满足幺半群的三大要求，即封闭性，结合律，和单元性。所以，Monad 的完整定义就是 “不过是自函子范畴里的幺半群”而已。这里的“不过是”很重要，因为永远存在的一个现象就是，不懂的人把这句话当天书，已经懂的人把这句话当废话。

Monad 的特性，绝不仅仅是在少写几行代码上面。它所具备的“顺序调用”，“短路操作” 的语义，达到的对“计算”的抽象所释放的表达能力，以及它结合律和封闭性带来的并发能力，都使得它成为一个十分强大的设计模式。不管是何种领域的程序编写，这个设计模式都必将是对已有工具库的一个有力补充。

当然，关于 Monad 还有一个传统就是，不看原始资料是永远懂不了的。 虽说是这样，个人还是希望未来会有一个不被看做是误导新人，却又能够显著降低理解心智负担的 Monad 资料的出现。
