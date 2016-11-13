---
title: 用 Ruby 实现 NFA
date: 2016-11-12 23:56:24
tags:
- compiling
- ruby
- regexp
---

Ruby 作为令人神往的一门语言，还是上手做点练习。本次实现重点在体验 Ruby 的表达性。

首先设计图的表达方式，由于 NFA 允许重复同权边，决定用邻边表来表示图，用起点节点来表示一个 NFA。一个节点的定义如下：

<!-- more -->

```ruby
  class State
    attr_reader :edges

    @@id = 0

    def initialize
      @edges = Hash.new { |hash, key| hash[key] = Set.new }
      @id = @@id
      @@id += 1
    end

    def link(input, dest)
      @edges[input] << dest
      self
    end

    def [](input)
      @edges[input]
    end

    def epsilon_closure
      met = Set.new
      cur = Set.new << self
      until cur.empty?
        met += cur
        cur = cur.map { |e| e[nil] }.reduce(Set.new, :+) - met
      end
      met
    end

    def inspect
      "[#{@id}]-#{@edges.to_a.inspect}"
    end
  end
```

出于便于跟踪的目的，临时给每个节点增加了 `id` 标识。用 `nil` 来表示输入为空的转移边。那么，一个点的 epsilon closure，就相当于该点沿着 `nil` 下的转移边能到达的点全体。实现时注意检查循环引用的情况就够了。

接下来实现 [Thompson-NFA](https://swtch.com/~rsc/regexp/regexp1.html), 也就是仅含最原始“拼接”、“或”以及“科林闭包”三种基本语义的正则表达语法。这块直接用传统的方式，没什么好讲的。

```ruby
class NFA

  attr_reader :head, :tail

  def initialize
    @head = State.new
    @tail = @head
  end
  
  def concat!(input, other = State.new)
    case other
    when NFA
      @tail.link(input, other.head)
      @tail = other.tail
    when State
      @tail.link(input, other)
      @tail = other
    end
    self
  end

  def or!(other)
    new_head = State.new
    new_head.link(nil, @head)
    new_head.link(nil, other.head)
    @head = new_head

    new_tail = State.new
    @tail.link(nil, new_tail)
    other.tail.link(nil, new_tail)
    @tail = new_tail
    self
  end

  def zero_or_more!
    new_head_tail = State.new
    new_head_tail.link(nil, @head)
    @tail.link(nil, new_head_tail)
    @head = new_head_tail
    @tail = new_head_tail
    self
  end
end
```

现在实现匹配的功能，匹配的过程可以看做从一个 epsilon closure set 沿着输入边转向下一个 epsilon closure set。注意这里面的 `set<vertex> -> [vertex] -> [set<vertex>] -> set<vertex>` 的转化过程，两个 flat_map 接一个 reduce。似乎嗅到了 Monad 的味道，唔。

```ruby
  def match(string)
    cur = @head.epsilon_closure
    string.each_char do |c|
      cur = cur.flat_map { |state| state[c].flat_map(&:epsilon_closure) }
               .reduce(Set.new, :+)
      return false if cur.empty?
    end
    cur.include?(@tail)
  end
```

我个人对于这个表达能力还是比较满意的，能从比较高的层次把算法描述清楚。而且基于上面文献，这样的实现效率还是能够保证的。

测试一下, 测试的 NFA 为 `(abc|abd)*`：

```shell
[83] pry(main)> g
=> #<NFA:0x007fe005752730
 @head=
  [0]-[[nil, #<Set: {[8]-[[nil, #<Set: {[0]-[["a", #<Set: {[1]-[["b", #<Set: {[2]-[["c", #<Set: {[3]-[[nil, #<Set: {[9]-[[nil, #<Set: {[0]-[[nil, #<Set: {...}>], ["a", #<Set: {}>]]}>], ["d", #<Set: {}>], ["a", #<Set: {}>]]}>], ["d", #<Set: {}>], ["a", #<Set: {}>]]}>], [nil, #<Set: {}>], ["e", #<Set: {}>], ["d", #<Set: {}>]]}>], [nil, #<Set: {}>]]}>], [nil, #<Set: {}>]], [4]-[["a", #<Set: {[5]-[["b", #<Set: {[6]-[["d", #<Set: {[7]-[[nil, #<Set: {[9]-[[nil, #<Set: {[0]-[[nil, #<Set: {...}>], ["a", #<Set: {}>]]}>], ["d", #<Set: {}>], ["a", #<Set: {}>]]}>]]}>], [nil, #<Set: {}>], ["e", #<Set: {}>], ["c", #<Set: {}>]]}>], [nil, #<Set: {}>]]}>], [nil, #<Set: {}>]]}>], ["a", #<Set: {}>]]}>], ["a", #<Set: {}>]],
 @tail=
  [0]-[[nil, #<Set: {[8]-[[nil, #<Set: {[0]-[["a", #<Set: {[1]-[["b", #<Set: {[2]-[["c", #<Set: {[3]-[[nil, #<Set: {[9]-[[nil, #<Set: {[0]-[[nil, #<Set: {...}>], ["a", #<Set: {}>]]}>], ["d", #<Set: {}>], ["a", #<Set: {}>]]}>], ["d", #<Set: {}>], ["a", #<Set: {}>]]}>], [nil, #<Set: {}>], ["e", #<Set: {}>], ["d", #<Set: {}>]]}>], [nil, #<Set: {}>]]}>], [nil, #<Set: {}>]], [4]-[["a", #<Set: {[5]-[["b", #<Set: {[6]-[["d", #<Set: {[7]-[[nil, #<Set: {[9]-[[nil, #<Set: {[0]-[[nil, #<Set: {...}>], ["a", #<Set: {}>]]}>], ["d", #<Set: {}>], ["a", #<Set: {}>]]}>]]}>], [nil, #<Set: {}>], ["e", #<Set: {}>], ["c", #<Set: {}>]]}>], [nil, #<Set: {}>]]}>], [nil, #<Set: {}>]]}>], ["a", #<Set: {}>]]}>], ["a", #<Set: {}>]]>
[84] pry(main)> g.match ''
=> true
[85] pry(main)> g.match 'abc'
=> true
[86] pry(main)> g.match 'abcabd'
=> true
[87] pry(main)> g.match 'ab'
=> false
[88] pry(main)> g.match 'abe'
=> false
[89] pry(main)> g.match 'abeabc'
=> false
```
