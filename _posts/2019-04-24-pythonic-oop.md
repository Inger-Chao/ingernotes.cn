---
title: "Pythonic OOP"
layout: post
date: 2019-04-24
tag: 
- Python
category: blog
author: ingerchao
description: The notes of python learning
---



Pythonic 的含义有两点：

1. 符合 Python 设计理念；
2. 写出更像 Python 原生的代码。 

#### 双下划线

##### 读法

- `_name` : single underscore name
- `__name`: double underscore name
- `__name__`：dunder name (double-under, 便于口头沟通创建的单词)
- `__init__()`：dunder init method (function)

##### 双下划线开头和结尾的变量或方法叫什么

- 类别：special；magic；dunder
- 实体：attribute；method

##### Special method

- **special method：**method with special name (dunder)
- **Why use it?**: A class can implement certain operations that are invoked by special syntax
- **original intention of design:** operator overloading

> Allowing classes to define their own behavior with respect to language operators.

#### 从语言设计层面理解 Python 的数据模型

##### 一切都是对象

- Python 中的**数据模型**是 `objects`;
- Python程序中**所有数据**都是用 `objects` 或者 `objects`之间的关系表示的；
- 甚至Python**代码**都是`objects`。

##### `objects` 的组成：identity

`objects`创建后，identity 再也不会改变直到被销毁。`id()` 和 `is`关注的就是一个`object`的 identity。

```python
a = 1.0
print(id(a))
a = 1.1
print(id(a))
a = 1.0
print(id(a))
b = 1.0
print(id(b))
c = 1.1
print(id(c))
```

```
4410044608
4410044656
4410044608
4410044608
4410044656
```

由以上代码可以看出，`1.0`本身拥有 identity，a 拿到的实际上是不可变对象`1.0`的 identity。

变量存的实际上是`object`的identity，创建出来的不同`object`有不同的 identity，变量的 id 变了并不是因为 `object`的identity改变，而是**变量存的`object`变了**。

对于不可变对象（` immutable object`），计算结果如果已经存在可直接返回相同的 identity。

##### `objects` 的组成：type

`objects`创建后，type 也不会改变直到被销毁。

`type()`函数返回一个 `object` 的type，type 表示对象属于哪个 class。type 决定了一个 object 支持哪些运算，可能的值在什么范围。

##### `objects` 的组成：value

- 可变对象（mutable object）value 可变；
- 不可变对象（immutable object）value不能改变；
- 当 `object` 是个 container 的情况，尤其需要注意可变和不可变。比如前面学到的 Tuple 中嵌套的 List。
- `Object`的type决定了它是否可变。

##### Container: 存放其他 Object reference 的 Object

当我们聚焦于Container的 values 时，我们关注的是value；当我们聚焦于Container的mutability时，关注的是identity。

#### Pythonic OOP with Speical Method and Attribute

```python
class X:
    pass
class Y(X):
    pass
def main():
    x = X()
    y = Y()
    # 输出对象的 class name
    print(x.__class__.__name__)
    print(y.__class__.__name__)
    # 输出类X所属类的 name 属性
    print(X.__class__.__name__)
    print(Y.__class__.__name__)
    # 输出对象父类所属类的name
    print(x.__class__.__base__.__name__)
    print(y.__class__.__base__.__name__)
    # 输出类父类所属类的name
    print(X.__class__.__base__.__name__)
    print(Y.__class__.__base__.__name__)
if __name__ == '__main__':
    main()
```

```
X
Y
type
type
object
X
object
object
```

可以看出每一个自定义的 class 都是一个 type object，每一个 class 在定义的时候如果没有继承，都会隐式继承 object 这个superclass。

- `object.__class__` ：对象的所属类
- `class.__name__`：类的name属性
- `class.__base__`：类的父类

#### Integrating Seamlessly with Python

```python
class X:
    pass
class Y(X):
    '''Class Y'''
    # 定义 3 个 special method
    def __str__(self):
        return "{} object".format(self.__class__.__name__)
    def __len__(self):
        return 10
    def __bool__(self):
        return False
def check_bool(x):
    if x:
        print("I'm {}. My bool value is True.".format(str(x)))
    else:
        print("I'm {}. My bool value is False.".format(str(x)))

def main():
    x = X()
    y = Y()
    print(x)
    # 隐式调用 __str__(self)
    print(y)
    # print(len(x)) 不注释会报错
    # 调用 __len__(self)
    print(len(y))
    check_bool(x)
    # 通过__bool__(self)修改 bool 的值
    check_bool(y)
    print(X.__doc__)
    print(Y.__doc__)
if __name__ == '__main__':
    main()
```

```
<__main__.X object at 0x7fcd741655f8>
Y object
10
I'm 0<__main__.X object at 0x7fcd741655f8>. My bool value is True.
I'm Y object. My bool value is False.
None
Class Y
```

- 之所以要实现 Special method，是为了让自定义的 class 与 Python 的内置函数无缝衔接；
- Python 有大量的内置函数，而这些函数大部分都是调用对象里的 special method。
- [Python 中所有的 special method](https://rszalski.github.io/magicmethods/)（大部分并不会用到）。

#### Attribute Access and Properties

Attribute 相关的一般操作是：Create、Read、Update 和 Delete。

> 数据库领域中的 CRUD 亦是如此。

```python
class X:
    pass

if __name__ == '__main__':
    # print(X.a)
    X.a = "a"
    print(X.a)
    X.a = "aa"
    print(X.a)
    del X.a
    # print(X.a)
```

- 默认情况下，CRUD 操作都支持，甚至说不用创建对象都可以做到（双下划线开头的除外）。
- 而如果对象中没有这个 Attribute，访问时会报错。