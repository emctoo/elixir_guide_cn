# 7 关键字(keywords)和映射(map)

迄今我们还没有讨论过关联数据类型，即那些把一个值（或多个值）和一个键值（key）关联起来的数据结构（译注：键值映射到一个或者多个值)。不同的语言对这些数据结构有不同的称呼，字典，哈希，关联数组等。

在Elixir中，我们有两种主要的关联数据结构：关键字列表和映射。是时候揭开它们的真面目了。

## 7.1 关键字列表

在许多的计算机语言中，经常用一个2元元组列表来表示关联数据结构。在Elixir里，当元组的列表中每个元组的第一个元素都是原子时，该列表称为关键字列表：


```
iex> list = [{:a, 1}, {:b, 2}]
[a: 1, b: 2]
iex> list == [a: 1, b: 2]
true
iex> list[:a]
1
```

从上面的例子中可以看到，Elixir支持使用特殊的语法来定义这种列表，而底层它就是一个元组列表。由于关键字列表本质上是列表，所有能用在列表上的函数也也适用于这种数据类型。例如，我们可以使用`++`把新的值加入到一个关键字列表中：

```
iex> list ++ [c: 3]
[a: 1, b: 2, c: 3]
iex> [a: 0] ++ list
[a: 0, a: 1, b: 2]
```

注意添加到前面的元素在查找的时候会被先找到:

```
iex> new_list = [a: 0] ++ list
[a: 0, a: 1, b: 2]
iex> new_list[:a]
0
```

关键字列表之所以重要，是因为它具有如下三种性质:

* 键必须是原子(atom)
* 键是有序的，该顺序有开发者指定
* 键可以重复出现

例如，[Ecto](https://github.com/elixir-lang/ecto)库利用上面的性质来，提供了一种优雅的DSL来编写数据库请求:

```
query = from w in Weather,
      where: w.prcp > 0,
      where: w.temp < 20,
     select: w
```

这些特性使得关键字列表成为了Elixir中给函数传递可选参数的默认机制。在第五章，当我们讨论宏`if/2`，我们提到如下的语法是合法的：

```
iex> if false, do: :this, else: :that
:that
```

`do:`和`else:`其实都是关键字列表！实际上，上面的写法等价于：

```
iex> if(false, [do: :this, else: :that])
:that
```

一般情况下，当关键词列表是函数的最后一个参数时，方括号是可选的。

为了能处理关键字列表，Elixir提供了[Keyword模块](http://elixir-lang.org/docs/stable/Keyword.html)。但是请记住，关键字列表本质上还是列表，因此它们的性能也是线性的。列表越长，查找元素、计算长度等花的时间越长。因此，关键字列表在Elixir中主要用于可选项(options)。如果你需要存储很多元素或者需要保证一个键最多映射到一个值，使用映射（map）。

尽管关键词列表上可以进行模式匹配，但是由于要求列表个数和顺序完全匹配，实际中很少使用。

```
iex> [a: a] = [a: 1]
[a: 1]
iex> a
1
iex> [a: a] = [a: 1, b: 2]
** (MatchError) no match of right hand side value: [a: 1, b: 2]
iex> [b: b, a: a] = [a: 1, b: 2]
** (MatchError) no match of right hand side value: [a: 1, b: 2]
```

## 映射（Maps）

在Elixir中，无论何时你需要存储一个键-值对，map都是最佳的数据结构选择。创建一个表单需要用`%{}`语法：

```
iex> map = %{:a => 1, 2 => :b}
%{2 => :b, :a => 1}
iex> map[:a]
1
iex> map[2]
:b
```

和关键字列表相比，我们可以清楚地看到两点不同：

*  表单允许任何类型的键
*  表单的键没有特定的顺序

和关键词列表相比，map和模式匹配结合之后非常有用。在模式中用到map时都会匹配到给定值的子集。

```
iex> %{} = %{:a => 1, 2 => :b}
%{:a => 1, 2 => :b}
iex> %{:a => a} = %{:a => 1, 2 => :b}
%{:a => 1, 2 => :b}
iex> a
1
iex> %{:c => c} = %{:a => 1, 2 => :b}
```

如上，只要模式中的的key出现在个定的map中就算是匹配。因此空map匹配所有的map。

可以在访问、匹配和添加map的key时使用变量:

```
iex> n = 1
1
iex> map = %{n => :one}
%{1 => :one}
iex> map[n]
:one
iex> %{^n => :one} = %{1 => :one, 2 => :two, 3 => :three}
```

Map模块提供了和`Keyword`模块相似的的API，以及各种函数用于操作map:

```
iex> Map.get(%{:a => 1, 2 => :b}, :a)
1
iex> Map.to_list(%{:a => 1, 2 => :b})
[{2, :b}, {:a, 1}]
```

当一个表单中的所有键都是原子的时候，你就可以用关键字语法了：

```
iex> map = %{a: 1, b: 2}
%{a: 1, b: 2}
```

map另一个有意思的特定是它提供了特有的语法用于更新和访问原子类的键：


```
iex> map = %{:a => 1, 2 => :b}
%{:a => 1, 2 => :b}
iex> map.a
1
iex> %{map | :a => 2}
%{:a => 2, 2 => :b}
iex> %{map | :c => 3}
** (ArgumentError) argument error
```

更新和访问都要求键必须存在。例如，最后一行之所以失败是因为在表单中没有键`:c`。

Elixir开发人员在使用map时，相较于Map模块中的函数，一般更喜欢 `map.field` 及模式匹配，以为后者体现出一种断定式的编程方式(assertive style of programming). 这篇博客给出了一些观点和例子，告诉你怎样通过断定式的Elixir代码，写出更简洁和高效的代码。

> 表单是新近才通过[EEP 443](http://elixir-lang.org/docs/stable/Map.html)被引入到Erlang虚拟机里的。Erlang 17提供了一个EEP的部分实现，它只实现了所谓的“小表单”。这意味着现在的表单只有当存储的元素数量不太大（几十个）的时候性能才有保证。为了弥补这一点，Elixir提供了[HashDict模块](http://elixir-lang.org/docs/stable/HashDict.html)，这个模块用一个哈希算法提供了一个支持存储几十万计键而又能保证良好性能的字典。

## 嵌套数据结构

有时候我们在map中又有map, 甚至在map中有关键词，等等之类。Elixir用 `put_in/2`，`update_in/2`和其他的宏，简化了嵌套数据结构的操作，提供了其他命令式语言一样的便利性，同时还保持了语言不变性这一特征。

假设有如下的结构:

```
iex> users = [
  john: %{name: "John", age: 27, languages: ["Erlang", "Ruby", "Elixir"]},
  mary: %{name: "Mary", age: 29, languages: ["Elixir", "F#", "Clojure"]}
]
```

这是个用户列表，值是姓名、年龄和喜欢的语言的一个map。如果想得到john的年龄:

```
iex> users[:john].age
27
```

恰好也可以用相同的语法更新值:

```
iex> users = put_in users[:john].age, 31
[john: %{name: "John", age: 31, languages: ["Erlang", "Ruby", "Elixir"]},
 mary: %{name: "Mary", age: 29, languages: ["Elixir", "F#", "Clojure"]}]
```

`update_in/2` 宏和这个相似，允许我们传入一个函数，用于控制值怎样修改。比如，想删掉Mary喜欢的语言中的"Clojure"：

```
iex> users = update_in users[:mary].languages, &List.delete(&1, "Clojure")
[john: %{name: "John", age: 31, languages: ["Erlang", "Ruby", "Elixir"]},
 mary: %{name: "Mary", age: 29, languages: ["Elixir", "F#"]}]
```

关于`put_in/2`和`update_in/2`还有更多东西需要了解，比如`get_and_update/2`可以一次性完成访问和修改。像`put_in/3`、`update_in/3`和`get_and_update_in/3`可以动态访问数据结构。查看`Kernl`模块中的他们各自的文档可以了解更多。

我们关于关联数据结构的介绍就是这些。可以看到，关键词列表和map是处理关系型数据结构的正确工具。
