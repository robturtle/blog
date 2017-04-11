---
title: About Coding Style
date: 2016-03-28 17:02:32
categories:
- chit-chat
tags:
- Java
- "code style"
---

Normally the preference of coding style is highly involved with one's aesthetics.  However, among all kinds of styling choices, there're still some principles which are universal and can be (sort of) reasoned with.  And two of the most important principles, would be *readability* and *compatibility*.  For readability, it's more like some organized arrangement optimization for human eye parsing.  For instance, inserting spaces between operators and identifiers.  For compatibility, it's more about some guarantees for identical appearance across platforms at source (sometimes also binary) level.  Typical examples are encoding, allowed charater set, etc.  Loosely speaking, most of the time when one's saying some code is "good looking", it's almost basically a mental reflection of "easily parsed".  Below I marked some typical styling decisions and tried to discuss their underlying logic without too much personal view.  

<!--more-->

## Indentation
The spec below is quoted from [Google Java Style](https://google.github.io/styleguide/javaguide.html):

> 2.3.1 Whitespace characters: the **ASCII horizontal space (0x20)** is the only whitespace characters that anywhere in a source file. This implies that:
>
> Tab characters are **not** used for indentation.

This is a request by compatibility.  The "unless" thing of not following it is "your code may look different from other machine".   Another point which is been talked over tens of thousands times is, the worse part of the violation is its invisibility for human eyes since spaces and tabs have the same appearance in almost all editors.  And if there're violated, the output from another machine with different tab configuration would be really ugly.

But how ugly it is?  Actually it's also correlated with another styling spec, line width.  Think about a smaller tab size is used natively but a larger one is used in a remote place.  The original line width compliant code would explode and generate maybe tons of line wraps in that remote machine.

Since the advantage is long discovered by the industry.  A convenient way of typing spaces using physical tab key is invented long ago, making the last advantage of using tabs uncompetitive.  Now we can see many programming languages are adopting whitespace as their only acceptable space character, this debate seems comes to its end.

> During the history, we will see a timeline that the configuration of tab size getting smaller and smaller. From 8, to 4, and ultimately, the "front-end style" 2.  Though I'm not saying that is a good evolution path, inherently we can see the development of the hardware, where higher resolution and richer GUI resources (like visible rulers) relieve the distinguishing responsbility from tabs.

## Separators

First of all, a well separated tokens are obviously good for human eye parsing.  Considering the Ruby code below:

```ruby
x=Pi*r**2+k/2>0?true:false
```

Now we insert some whitespaces:

```ruby
x = Pi * r ** 2 + k / 2 > 0 ? true : false
```

Much better. But does it the best way for this expression? Considering its math form:

$$
x = \pi \cdot r^2 + \frac{k}{2} > 0 \,? \text{ true} : \text{ false}
$$

Apparently is better than the code form.  This is because the code form flatten the information of priority.  The almost toppest priority operator `/` sits just right next to the lowest priority operator `? :`, making it looks like all the operations are processed in sequence.  This forces an additional process for filling in the priority information when a human is reading it. The code will be much worse when increasing the number of priority level.

Normally we got some trick to help human brain parse.  The first one is adding parentheses:

```ruby
x = ((Pi * r ** 2) + (k / 2) > 0) ? true : false
```

I do appreciate another approach, which we don't separate it taking tokens as unit but a larger logical group:

```ruby
x = Pi*r**2 + k/2 > 0 ? true : false
```

Where `Pi*r**2` is a common sense well known to most people, which is perfect to be grouped as one unit.  Similar situation goes with `k/2`.  Hence we express a better solution without introducing parentheses.

Why not introducing parentheses is important?  Cause parentheses are not human brain friendly.  For my personal experience, it's extremely hard to understand an expression with 3+ nested parentheses.  A typical scenario for that is the C function pointer syntax.

> For Lisp, I highly doubt its readability before the rainbow parentheses were invented.

The drawback of the latter approach is obvious --- it can only support two level hierarchy.  This means it can also do little to highly nested expression.  To deal with that, some uses the number of whitespaces to indicate the nested level:

```ruby
x = Pi*r**2 + k/2 > 0  ?  true  :  false
```

I will not comment on this approach.  But keep in mind that besides all the ways discussed above, we always have a simple yet powerful solution --- split the expression.

## Inside parentheses
Most old styles require no spaces inside parentheses:

```
compare(a, b);
```

Not know since when, a with-space style somehow became famous around certain groups.  I personally first saw such kind of code in a Java program.

```
compare( a, b );
```

The reasoning is as above --- separation is good for human brain.  Since in most compact arranged code I always see function name and its arguments sticking together, I will refer the separation inside the parentheses brilliant!  A difference is for the no-space style I will almost need more effort to going into the details before I can understand a line of code, but for the with-space style, I can easily grasp the whole construct of a block of code.

In spite of its good part, problem raised --- should we also adding spaces inside the nested parentheses?  Considering the expression below:

```
Math.sqrt(Math.pi, Math.pow(Math.abs(a - findNeg(b)), Math.pow(2, findNeg(b))))
```

The all space inserted version is like this:

```
Math.sqrt( Math.pi, Math.pow( Math.abs( a - findNeg( b ) ), Math.pow( 2, findNeg( b ) ) ) )
```

The effect of the separation is indeed weakened due to the deep nesting.  The only consequence is it constructed a loose looking.

In addition, the tradeoff in using inside space is also related to the color schema used in the editting environment.  A high contrast schema would be a better replacement over separation with only one problem, compatibility issues.

## Semicolons

The problem of semicolons is the nature of its asymmetry.  Taking a list definition as an example:

```ruby
foods = [
  apple,
  banana,
  orange
]
```

By syntax there is no need to put a semicolon after the last element.  It's correct but inconvenient.  Considering now you're about to appending another element `peer` into that list, you will have to append a semicolon in the line of `orange` first.  The inherent problem is actually related to the `new/delete` problem in C++ --- any non-atomic operation exposed to human is doomed to causing errors.  Especially at the era when developing environment doesn't got enough resources to do realtime checking.

A hack for this inconvenience is by grammar definition that, we allow the trailling semicolon in a literal list expression. Sounds fine but the thing is there're many other scenarios where this rule cannot be applied.  Say variable declaration.

A solution for that is aligning semicolon with the type identifier:

```c
int a = 1
  , b = 2;
```

Another example is C++ initialization list.  The solution is also alignment.

```c++
Constructor()
: para1(1)
, para2(true)
, para3(3)
{
  /* ... */
}
```

By doing so, whenever we want to append/insert one element, we will only need to change one line of code.  At least, we made the existance of semicolons notable.

I will say it is a great solution.  The only problem is its lacking of tool supports.  Most of the auto formatting tools in pervaling editors support it badly (Same thing happened to the inside space style).  I call it a tool-level compatibility issue.
