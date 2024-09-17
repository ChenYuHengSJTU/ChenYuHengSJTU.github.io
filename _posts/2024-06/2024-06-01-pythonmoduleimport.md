---
title: 'Python导入自己创建的包、模块'
date: 2024-06-01
permalink: /posts/2024/06/pythonmoduleimport/
tags:
  - python
  - solutions
excerpt: ""
---

<!-- more -->

## 模块、脚本、 包
首先明确几个概念，脚本、模块、包

### 模块
模块就是一系列python对象的集合，包括类的定义、函数、常量等，与C++中的hpp或h文件类似，（尽管有些可以运行）但是不被视作可以运行的文件。

python官方手册中对于模块的解释如下：
>模块是包含 Python 定义和语句的文件。其文件名是模块名加后缀名 .py 。在模块内部，通过全局变量 __name__ 可以获取模块名（即字符串）

### 脚本
脚本是解释器解释运行的文件，通常会使用if \_\_name\_\_ == "\_\_main\_\_":来运行主函数。与模块不同的是，无论脚本的名字如何，其__name__属性始终为__main__，**即解释器将脚本作为视作__main__.py文件运行**。

### 包
包一个层级嵌套式的文件（夹），可以类比操作系统中的文件夹进行理解。包是python为了帮助组织模块并提供名称层次结构。要注意的一个重点概念是所有包都是模块，但并非所有模块都是包。 或者换句话说，包只是一种特殊的模块。 特别地，任何具有__path__属性的模块都会被当作是包。

对于包，目录下必须含有__init__.py文件，以表示当前目录是一个python包，\_\_init\_\_.py文件可以留空。

### 包相对导入
在包这里，包相对导入需要特别强调一下，包相对导入和linux下文件的相对路径类似，下面是python官方手册的解释：

>相对导入使用前缀点号。 一个前缀点号表示相对导入从当前包开始。 两个或更多前缀点号表示对当前包的上级包的相对导入，第一个点号之后的每个点号代表一级。

## 运行python文件
这里不使用特定的IDE，使用命令行环境运行python文件（脚本）。

使用命令行运行python脚本的方式如下：
```bash
python [-bBdEhiIOqsSuvVWx?] [-c command | -m module-name | script | - ] [args]
```
### 参数
python有几个常用的命令行参数，如-c，-m，-I，-E等，以及不适用任何命令行参数运行脚本我呢间。这里主要讨论两种方式，即-m运行模块和直接运行脚本。

#### -m
使用-m参数运行python模块，文件（夹）之间的层属关系不再使用/或\，而是使用英文句号.，并且不能添加.py文件后缀名。

**需要特别注意的是，使用-m运行模块，与-c选项一样，当前目录将被加入环境变量sys.path的开头。**

对于模块，我们可以直接使用
```bash
python -m mod1.mod11.mod111
```
来运行mod1目录下的mod11子目录中的mod111.py模块。

如果我们希望运行一个文件夹，比如上述的mod1或者mod11，我们也可以使用-m命令行参数运行。但是该包必须包含一个__main__.py文件，运行该包时，实际上python解释器运行的是其包含的__main__.py文件。以下是python官方文档中的说明：

>包名称（包括命名空间包）也允许使用。使用包名称而不是普通模块名时，解释器把 <pkg>.__main__ 作为主模块执行。此行为特意被设计为与作为脚本参数传递给解释器的目录和 zip 文件的处理方式类似。

#### python script.py
如果我们不适用-i或者-m参数，直接运行脚本（或者加入）-bBdEhiIOqsSuvVWx?等选项和一些参数argv。

同样，我们可以使用python dir的方式运行一个文件夹，并且解释器会运行该文件夹下的__main__.py文件。

**需要特别注意，这种运行方式会将文件所在目录加入sys.path的开头。(与-m区分开)**

## 实例
下面使用一个实例来讨论几个问题。

在linux环境下构建如下文件夹结构：
```text
parent
├── __init__.py
├── mod1
│   ├── __init__.py
│   ├── mod11
│   │   ├── __init__.py
│   │   └── mod111.py
│   └── mod12.py
├── mod2
│   ├── __init__.py
│   ├── __main__.py
│   ├── mod21.py
│   └── mod22.py
└── mod3.py
3 directories, 10 files
```
在parent文件夹内，我们将围绕mod21.py进行讨论。

mod21.py的内容如下
```python
print(__name__)
print(__spec__)
import sys
print(sys.path)
from mod22 import helloworld22
from mod1.mod12 import helloworld12
from ..mod1 import *
from ..mod1.mod12 import helloworld12
from ..mod1.mod11.mod111 import helloworld111
from ..mod3 import helloworld3
from .mod22 import helloworld22
 
helloworld12()
helloworld111()
helloworld3()
helloworld22()
```

 三个包中的__init__.py的内容如下（以mod1中为例，其余类似）

```python
if __name__ == "__main__":
    print("mod1 runs as main programme")
else:
    print("mod1 runs as a package")
```

### parent/mod2/目录下执行python mod21.py
此时报错ModuleNotFoundError: No module named 'mod1'

mod22可以正确导入，因为在同一级目录下

但是为什么mod1没有正确导入呢？请往下看~

### parent/目录下执行python mod2/mod21.py
同样报错ModuleNotFoundError: No module named 'mod1'

这是因为无论在哪个文件夹（$pwd）内运行mod21.py，解释器向sys.path中添加的，始终都是parent/mod2/，即mod21.py所在的目录，所以无法看到mod1。

**解决方案1**：使用sys.path.append导入mod1或者parent的路径。注意，如果导入mod1的路径，那么就无需from mod1 import ...了，直接导入mod1下面的模块。

**解决方案2**：使用环境变量PYTHONPATH，其表示增加模块文件默认搜索路径。 所用格式与终端的 PATH 相同。运行mod21.py前，使用export设置环境变量PYTHONPATH

```bash
export PYTHONPATH=parent
```

随后mod1可以正确导入，且输出mod1 runs as a package

但是接着报错

>Traceback (most recent call last):  
  File "parent/mod2/mod21.py", line 8, in <module>  
    from ..mod1 import *  
ImportError: attempted relative import with no known parent package  
​
..mod1代表我们使用了包相对导入，但是这里为什么无法工作呢？

在[6. 模块 — Python 3.12.3 文档](https://docs.python.org/zh-cn/3/tutorial/modules.html#packages)中，有这样一段话

>注意，相对导入基于当前模块名。因为主模块名永远是 "__main__" ，所以如果计划将一个模块用作 Python 应用程序的主模块，那么该模块内的导入语句必须始终使用绝对导入。

我们现在对此做详细解释。

可以看到，mod21.py中第二行我们加入了一句print(\_\_spec\_\_)。\_\_spec\_\_是\_\_main\_\_.py文件中所独有的属性。这里我们通过测试，发现输出为None。只有当附加-m选项时，\_\_spec\_\_ 会被设为相应模块或包的模块规格说明。其余情况下均为None。正因为我们没有使用-m选项，所以这里的__spec__为None，代表mod21.py中代码不直接与可导入的模块相对应，即与其他.py文件不再具有包内相对位置的关系。所以我们无法使用**包相对导入。只能使用绝对导入，即将mod1添加到sys.path中。**
​
### parent/目录下运行python -m mod2.mod21

>Traceback (most recent call last):  
  File "parent/mod2/mod21.py", line 5, in <module>  
    from mod22 import helloworld22  
ModuleNotFoundError: No module named 'mod22'  

注意，使用-m选项，此时sys.path中添加的是命令行的工作环境$pwd。但是此时报错无法引入同一级中的mod22。注意到此时__spec__不再为None，而是

>ModuleSpec(name='mod2.mod21', loader=<_frozen_importlib_external.SourceFileLoader object at 0x7f15e7b4db50>, origin='/mnt/d/Tmp/python/mod2/mod21.py')

 那么我们使用绝对引用的方式导入mod22当然不成功，只能使用相对导入的方式。

我们将这两行绝对导入的语句去掉，运行，结果还是报错

>Traceback (most recent call last):
  File "parent/mod2/mod21.py", line 8, in <module>
    from ..mod1 import *
ImportError: attempted relative import beyond top-level package

 从报错结果来看，无法导入mod1是因为相对导入的层级超过了当前包的层级，我们需要退回到parent的父级文件夹中运行

### parent父级目录下运行python -m parent.mod2.mod21
此时运行正确，完整的输出为

>parent runs as a package  
mod2 runs as a package mod2  
__main__  
ModuleSpec(name='python.mod2.mod21', loader=<_frozen_importlib_external.SourceFileLoader object at 0x7f6e3ca7bb80>, origin='/parent/mod2/mod21.py')  
['parent/..',]  
mod1 runs as a package  
python.mod1.mod12  
Running as a package mod11  
python.mod1.mod11.mod111  
hello world  
Hello World mod12  
Hello World mod111  
python.mod2.mod22  
Hello World mod12  
Hello World mod111  
Hello World mod3  
Hello World mod22  

## 参考
参考python文档：[5. 导入系统 — Python 3.12.3 文档](https://docs.python.org/zh-cn/3/reference/import.html#special-considerations-for-main),[1. 命令行与环境 — Python 3.12.3 文档](https://docs.python.org/zh-cn/3.12/using/cmdline.html#environment-variables)