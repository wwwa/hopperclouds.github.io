title: NotImplemented
date: 2016-10-09 16:00:00
categories:
  - 技术
tags:
  - Python
  - NotImplemented
author: Kxrr

---

在创建基类时常常会用到`raise NotImplementedError`这个语句, 但在写下这条语句时IDE可能会补全一个`NotImplemented`出来, NotImplemented是什么?

<!--more-->

## NotImplemented是什么

首先NotImplemented并不是一种异常, 而是Built-in的一种类型:

```python
>>> type(NotImplemented)
<type 'NotImplementedType'>
```

官方文档中是这么描述的:

> Special value which can be returned by the “rich comparison” special methods (\_\_eq\_\_(), \_\_lt\_\_(), and friends), to indicate that the comparison is not implemented with respect to the other type.

## NotImplemented的具体应用

根据文档描述, NotImplemented常用在object.\_\_eq\_\_这样的比较方法中。

在下面的例子中, 比较Pants和Socks对象时, 首先会调用Pants的\_\_eq\_\_方法, 返回的是`NotImplemented`则转而调用Socks的\_\_eq\_\_方法。

使用NotImplemented而不是抛出异常, 给了其它对象扩展的机会。

```python
class Entity(object):
    def __init__(self, size):
        self.size = size


class Pants(Entity):
    def __eq__(self, other):
        return NotImplemented


class Socks(Entity):
    def __eq__(self, other):
        if not isinstance(other, self.__class__):
            return False
        return self.size == other.size


if __name__ == '__main__':
    print Pants(5) == Socks(5)
```

---

文末留一个小问题:

```python
class Foo(object):
    def __lt__(self, other):
        return NotImplemented
```

`Foo() < Foo()`有输出吗? 如果有, 是什么？


