---
layout: post
title: "《Python源码剖析》读书笔记"
category: Note
tags: Python
---

从Python开始，向动态语言迈进。所幸刚开始就接触了这本书，让我对虚拟机、字节码、垃圾回收机制、内存池等技术有了最直观的认识。并且有了用c 编写高级语言的实现框架，不得不佩服c 的强大。读完这本书后，再看《Python Tutorial》就简单多了。以下是关于本书内容的一些笔记及个人理解。

## Python的内建对象

Python 中一切都是对象，每个类型都是 `PyObject` 的子类。每个类型中至少包含两个属性：类型信息`ob_type` (动态创建)，和引用计数器`ob_refcnt` (垃圾回收)。Python 通过内部的`PyXXX_Type` 对象创建对象。

<!-- more -->

### Python 的对象从概念上大致分为5类：
- Fundamental 对象： type
- Numeric 对象： int、float、boolean
- Sequence 对象：string、list、tuple
- Mapping 对象：dict
- Internal 对象：function、code、frame、module、method.

1. Python中的整数对象

Python内部用PyIntObject表示整数对象，PyIntObject对象是不可变对象，因此每一个PyIntObject 对象都能够被任意的共享。在内部通过PyInt_Type进行创建，对于小整数(Python 2.5 默认的小整数范围为[-5, 257)，可调整) 直接将其所对应的PyIntObject对象缓存在内存中，防止频繁分配内存。而对于其它整数，Python 运行环境使用空闲链表管理，每次分配一个block，由其它大整数轮流使用。当没有空闲的空间时继续分配，使用完后返回给空闲链表。分配的内存在Python结束之前，永远不会被释放。由于内存共享，Python用于实现整数对象的内存池与历史上创建的整数对象的个数无关，而仅仅与同一时刻共存的整数对象个数的最大值有关。

2. Python中的字符串对象

Python中的字符串对象使用PyString_Type创建，用PyStringObject表示。该对象使用ob_size表示字符串大小，中间可以有'\0'，ob_sval 指向字符串的首地址。Python使用interned机制管理已分配的String，并加入缓冲池中。Python中通过\'+\'号连接字符串效率低下，如果要连接N个PyStringObject对象，则要进行N-1次内存申请及搬运工作，所以尽量使用append或者join方法。

3. Python中的List对象

List对象是一个可变对象，支持插入、删除等操作。在Python中的列表中存放的是PyObject*对象，可将PyListObject 看成 vector PyObject;。PyListObject所采取的内存管理策略和C++中的vector采取的策略也很相似，先分配一大块内存，使用完后再申请。

4. Python中的Dict对象

Python的内部实现使用了大量的Dict，Dict是关联式容器，使用(key, value)表示。关联式容器设计时总会极大的关注键的搜索效率，C++中的关联容器map采用RB-tree实现。而Python对搜索效率要求极其苛刻，采用了散列表，在最优情况下，搜索的复杂度为O(1)。

## Python的编译过程

Python编译程序和其它语言编译程序一样，需要经过以下几步：

1. Tokenizer进行词法分析，把源程序分解为Token
2. Parser根据Token创建CST(Concrete Syntax Tree)
3. CST被转换为AST(Abstract Syntax Tree)
4. 将AST编译成字节码
以下是Python中`print "Hello World!"` 生成的字节码：

```python

>>>source = open('demo.py').read()
>>>co = compile(source, 'demo.py', 'exec')
>>>import dis
>>>dis.dis(co)
1             0 LOAD_CONST               0 ('Hello World!')
              3 PRINT_ITEM
              4 PRINT_NEWLINE
              5 LOAD_CONST               1 (None)
              8 RETURN_VALUE

```

## Python的虚拟机

PyCodeObject 对象中存着Python源码经过编译后的Bytecode序列。每个Python的源文件按名字空间可生成多个PyCodeObject 。在import 加载一个 demo之后，编译结果会被存在.pyc文件中，下次调用时直接从pyc文件中读到PyCodeObject。Python虚拟机运行时按字节码解释程序，调用内建对象执行操作，返回结果。Python中使用PyFrameObject模拟栈帧，负责对字节码的解释和函数参数的传递。

## 垃圾回收机制

主流的垃圾回收技术有：标记--清除、停止--复制等方法，Python内部设计时使用引用计数机制进行垃圾回收。出现循环引用的对象无法被引用技术机制回收，Python使用标记--清除方法，消除循环引用。

垃圾收集机制一般分为两个阶段：垃圾检测和垃圾回收。标记--清除方法同样遵循垃圾回收的两个阶段，其简要工作如下：

1. 寻找根对象集合。
2. 从根对象出发，沿着跟对象，将对象集合分为可达对象和不可达对象，使用三色标注法。[垃圾检测]
3. 回收所有不可达对象。[垃圾回收]

### 分代回收

一系列的研究表明：一定比例的内存块的生命周期都比较短，而剩下的内存块，其生存周期会比较长，甚至会从程序开始一直持续到程序结束。分代垃圾回收机制将系统中的所有内存块根据其存活时间划分为不同的集合，每一个集合就是一个代。垃圾收集的频率随着代的存活时间的增大而减少。

