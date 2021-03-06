# 1. 基础
## 1.1 python包安装的路径
`sudo apt-get install`，`pip install`，`sudo pip install`，`pip install –user`，`pip3 install`，`sudo pip3 install`，`pip3 install –user`几种安装方式的安装路径不同。  
* 其中pip和pip3主要是python版本的区别，下表只以`pip`为例。   
  * `pip3`时，将路径中的`python2.7`换成`python3.5`即可。   
  * `dist-packages`是从包管理器安装的python包的路径； `site-packages`是从第三方库安装的的python包的路径

   | 命令 | python包安装路径 | 说明 |
   | :------------ |:---------------|:---|
   | sudo apt-get install | /usr/lib/python2.7/dist-packages |apt-get install必须要sudo|
   | sudo pip install | /usr/local/lib/python2.7/dist-packages |用sudo可能会报错：cannot import name 'main'|
   | pip install | ～/.local/lib/python2.7/dist-packages |可能会报错： Permission denied|
   | pip install --user| ～/.local/lib/python2.7/dist-packages |--user 使包安装在当前用户文件夹下|

* 在.py中输入
  ```python
  import sys,pprint;  
  pprint.pprint(sys.path)   # 输出一个换一行
  ```
  可查看环境变量，环境变量顺序对应于包的导入顺序
* 假设虚拟环境路径为`~/venv`， python3虚拟环境中安装的包，可以在`~/venv/lib/python3.5/`中找到，但这些包只能在该虚拟环境下调用
  * 例如要想在`jupytor-notebook`中能import在venv中安装的包，`jupytor-notebook`本身也需要安装在venv内

## 1.2 python虚拟环境
> 参见： [虚拟环境是什么?](https://ningyu1.github.io/site/post/63-python-virtualenv/)  

* 虚拟环境(venv)工具，可以避免不同项目所需的python版本不同而引起的问题  

    ___在默认情况下，所有安装在系统范围内的包对于venv都是可见的，在venv中安装的包，就只能在venv中用。 当没有在venv找到对应软件包时，还是会全局环境找。___ 例如对于Ubuntu16.04， python3的venv只代表优先使用python3， 在python3的包没找到的时候还是会找python2的包， 用venv可实现python2、 3的的混合编程。    
    以上这种行为可以被更改：在创建venv时增加`--no-site-packages`选项，venv就不会读取系统包，会一个完全独立和隔离的Python环境。  
    ```bash
    # Install
    sudo pip3 install -U virtualenv

    # Create a new virtual environment by choosing a Python interpreter and making a ./venv directory to hold it:
    virtualenv --system-site-packages -p python3 ./venv

    # Activate the virtual environment using a shell-specific command:
    source ./venv/bin/activate  

    # Check the installation
    pip install --upgrade pip
    pip list
    python --version        # 显示3.x， 也即虚拟环境下只安装了python3
    python3 --version       # 显示3.x
    python2 --version       # 还是会显示2.x。 Python3虚拟环境没找到包，还是会去Python2的环境里面找

    # Quit the vitual env
    deactivate              
    python --version        # 显示2.x
    ```

* 对于python环境而言，`pip + venv` 比 `Anaconda` (https://blog.csdn.net/lwgkzl/article/details/89329383) 稳定，Anaconda出现一些奇奇怪怪的毛病

## 1.3 慎用相对路径
* 在.py文件中，相对路径（./等）始终是相对於終端本身的，进行操作有机会报错。 例如在`jupyter-notebook`使用，显示`/home/alex/.local/lib/python3.5/site-packages`，这个是`jupyter-notebook`的安装目录。
解决方案参见： https://www.jianshu.com/p/76a3d317722c
    ```python
    import os, sys
    print( os.path.split(os.path.realpath(sys.argv[0]))[0], os.path.dirname(__file__) )    # 这两个都是文件所在目录
    print('realpath', os.path.realpath(sys.argv[0]))      # 文件的绝对路径
    ```
* `os.path.expanduser（～/Desktop/a.txt` 可将路径转换为根目录下的绝对路径

## 1.4 import
### 1.4.1 相对导入和绝对导入  
> https://medium.com/@alan81920/python-import-%E7%B0%A1%E6%98%93%E6%95%99%E5%AD%B8-c98e8e2553d3  
> https://blog.csdn.net/u010138758/article/details/80152151

相对导入和绝对导入：

| 导入方式 | 相对导入 | 绝对导入 |
| :------------ |:---------------|:---|
| 格式 | `from .A import B` 或 `from ..A import B`。其中`.`代表当前模块，`..`代表上层模块，依次类推（或需要配合`python -m`使用)|`import A.B` 或 `from A import B`|
| 优点 | 相对导入可以避免硬编码，更改包名后，代码仍然可以运行，可维护好 | 可读性好。绝对导入可以避免与标准库命名的冲突，实际上也不推荐自定义模块与标准库命令相同。|
| 以哪个目录为参考目录 | import语句所在的文件，其所属package的目录 |当前工作目录（例如执行命令 `python3 a.py`时，文件`a.py`所在的目录）|
|要求| 必须要在package内，所谓的package，就是包含 `__init__.py` 文件的目录 | 有没有package都可以使用|


### 1.4.2 python import 的规则
python在import时，可以按三种方式找：  
* 按绝对导入方式查找包
* 有 package 时，按相对导入方式查找包
* 在 python 安装路径的 lib 库中（例如`/usr/lib/python3.X`）  

### 1.4.2 python import 的例子

```
-- src
    |-- train.py: from utils.dataset import * 
    |-- utils
        |-- __init__.py
        |-- dataset.py
        |-- utils.py
```
当前工作目录： `/src`,  
命令行输入：`python3 train.py`,  
想要在`dataset.py`中 `import utils.py`的函数，有两种方法： 
* 使用绝对导入：`from utils.utils import *`，使用`from utils import *` 是行不通的，因为没有`/src/utils.py`) 
* 使用相对导入：`from .utils import *`

### 1.4.3 其他
* import 其他文件夹下的module  
  > http://www.361way.com/python-import-dif-dir-module/4064.html  
  > https://medium.com/@alan81920/python-import-%E7%B0%A1%E6%98%93%E6%95%99%E5%AD%B8-c98e8e2553d3 

  程序结构：  
    ```
    -- src
        |-- __init__.py
        |-- main.py: from pkg_a.a import all
        |-- a
            |-- __init__.py
            |-- a.py
        |-- test
            |-- __init__.py
            |-- test.py
    ```
    当前工作目录： `/src`,  
    命令行输入：`python3 main.py`  
    如果`a.py`中想要引入`test.py`，需要的做法如下：  
    ```python
    import sys
    sys.path.append("..")
    from test.test import *
    ```
    直接使用 `from ..test.test import *` 会导致报错：`attempted relative import beyond top-level package`。是因为`src`本身这个 package 并没有被记录下来，见：https://stackoverflow.com/questions/30669474/beyond-top-level-package-error-in-relative-import

* \_\_all__ 和 import * 
  ```python
  ### 文件foo.py ###
  __all__ = ['x', 'test']
  x = 2
  y = 3
  def test():
      print('test')
  ```
  ```python
  ### 文件main.py ###
  from foo import *
  print('x: ', x)
  print('y: ', y)     #此句会报错，因为没有导入y
  test()

  # `import *`表示导入import all；但`__all__`指定了能够被导入的部分（例如 y 就没有被包含）
  ```

* from \_\_future__ import  

    用于导入之后版本的一些功能。  
    例如，在开头加上`from __future__ import print_function`这句之后，即使在python2.X，
    print就可以像python3.X那样加括号使用（python2.X中print不需要括号，而在python3.X中则需要） 


## 1.5 \*args和\*\*kwargs
```python
def test(a,*args,**kwargs):
    print a
    print args
    print kwargs

test(1,2,3,d='4',e=5)
# 输出结果：
# 1
# (2, 3)
# {'e': 5, 'd': '4'}
 ```
意思就是1还是参数a的值，args表示剩余变量的值，kwargs在args之后表示成对键值对。

## 1.6 if \_\_name__ == '\_\_main__':
每个python模块（python文件）都包含一些内置的变量，其中：
* `'__main__'`等于当前执行文件的名称（包含后缀.py)  
* `__name__`
    * 当运行模块被执行的时候，`__name__`等于文件名（包含后缀.py）
    * 当作为模块被import到其他文件中时，则`__name__`等于模块名称（不包含后缀.py）
    
所以，当.py文件被直接运行时，`if __name__ == '__main__'`为真；当.py文件被作为模块导入其他文件时，`if __name__ == '__main__'`为假。


## 1.7 扁平结构比嵌套结构好
 ```python
import numpy as np
a = np.array([-1,2,3,-4,5])
a = [x+3 if x<0 else x for x in a]  # 得到[2, 2, 3, -1, 5]
```

## 1.8 继承object类
见： https://www.zhihu.com/question/19754936  

属于历史遗留问题，继承 object 类的是新式类，不继承 object 类的是经典类。在Python3中已经不存在这个问题，object已经作为所有东西的基类了。

## 1.9 异常处理
> 见： https://blog.csdn.net/u012609509/article/details/72911564  

`with` 和 `try except`功能不同，with语句只是帮忙关闭没有释放的资源，并且抛出异常，但是后面的语句是不能执行的。为了即能够输出我们自定义的错误信息，又能不影响后面代码的执行，必须还得使用`try except`语句

* with
    ```python
    with clause as B:
        do_something()
    ```

    * 紧跟 with 后面的语句 clause 被求值后，其返回值是一个对象，称它为A。该对象的`–enter–()`方法被调用，`–enter–()`方法的返回值将被赋值给as后面的变量 B
    * 当 `do_something()` 全部被执行完之后，将调用 A 的`–exit–()`方法，其会自动释放资源。
    * 在 with 后面的代码块抛出异常时，`exit()`方法也被执行。 

* try except
    > https://stackoverflow.com/questions/18675863/load-data-from-python-pickle-file-in-a-loop  
    ```python
    # 不用担心pkl.load()已经读取完毕报错
    import pickle as pkl
    def pickleLoader(pklFile):
        try:
            while True:
                yield pkl.load(pklFile)
        except EOFError:
            pass

    with open(filename) as f:
        for event in pickleLoader(f):
            do_something()
    ```


---
<br>

# 2. 方法
## 2.1 super()和__call__()方法
> https://www.zhihu.com/question/20040039
* super(): 使用继承时，基类的函数不会自动被调用。需要手动调用，例如调用基类的构造函数：`super().__init__()`  
    * Python3.x 和 Python2.x 语法有区别: Python2 中用`super(Class, self).xxx`，Python3 中简化了，可以用`super().xxx`  
    * 多继承时， `super(A, self).func` 执行的是MRO中的下一个类的 func；MRO 全称是 Method Resolution Order，它代表了类继承的顺序。老老实实地用类名去掉就不会出现这种问题。
        ```python
        class C(A,B):
        def __init__(self):
            A.__init__(self)
            B.__init__(self)
        ```
    * `self` 是首先调用自身的方法如果自身没有再去父类中，`super` 是直接从父类中找方法
* \_\_call__()： 如果在创建class的时候写了`__call__()`方法，那么该class实例化出实例后，实例名()就是调用`__call__()`方法。`__call__()`方法使实例能够像函数一样被调用。
```python
# Python2 代码
class Person(object):
    def __init__(self, name, gender):
        self.name = name
        self.gender = gender
    def __call__(self, friend):  
        print 'My name is %s...' % self.name
        print 'My friend is %s...' % friend

# 定义Student类时，只需要把额外的属性加上，例如score：
class Student(Person):
    def __init__(self, name, gender, score):
        super(Student, self).__init__(name, gender)  # 若不写这一句，子类将缺失name和gender属性
        self.score = score

p = Person('Bob', 'male')
p('Tim')        # 调用了__call__方法
# 输出结果：
# My name is Bob...
# My friend is Tim...
```

## 2.2 enumerate()方法
对于一个可遍历的对象（如# 分别用 loc[0][0], loc[0][1] 获取第一个在第一、第二维上的位置列表、字符串），enumerate()方法将其组成一个索引序列，利用它可以同时获得索引和值。
```python
list1 = ["a", "b", "c"]
for index, item in enumerate(list1, 1):    # 第二个参数表明从1开始索引
print index, item
# 输出结果
# 1 a
# 2 b
# 3 c
```
## 2.3 一些处理文本的方法
|方法|用途|
| :------------ | :-----|
|split()|将字符串按空格分割，返回分割后的列表|
|lstrip(), rstrip()|去掉左右空格|
|join()|`str = "-"; seq = ("a", "b", "c"); print(str.join(seq));`>>>`a-b-c`|
|repr()|输入为对象，返回一个对象供解释器读取的 string 格式|
|str.format()|使字符串格式化，例如`{1} {0} {1}".format("hello", "world")`>>>`'world hello world'`|

---
<br>

# 3. 其他特性
## 3.1 多变量赋值
```python
a, b = b, a        # 交换a，b

# 慎用连等，这会使得a,b,c都是同一个对象的引用。改变其中一个，其他两个也会被改变
a = b = c = 1    
a, b, c = 1, 2, "john"
```  

## 3.2 生成器和迭代器
### 3.2.1 生成器
* 为什么要有生成器？  

    固然很多时候，我们可以直接创建一个列表，但受到内存限制，列表容量肯定是有限的。而且创建一个包含100万个元素的列表，不仅占用很大的存储空间，而且如果我们仅仅需要访问前面几个元素，那后面绝大多数元素占用的空间都白白浪费了。所以，如果列表元素可以按照某种算法推算出来，那我们是否可以在循环的过程中不断推算出后续的元素呢？这样就不必创建完整的list，从而节省大量的空间，在Python中，这种一边循环一边计算的机制，称为生成器：generator。  
    
    生成器一次只能产生一个值，这样消耗的内存数量将大大减小，而且允许调用函数可以很快的处理前几个返回值，因此生成器看起来像是一个函数，但是表现得却像是迭代器。

* 生成器是什么？  

    ***任何使用了yield的函数都称为生成器，调用生成器函数将创建一个对象，该对象有一个`__next__()`方法。***    
    
    例如下面的函数spam（）是一个生成器，对spam（）的调用不会执行函数里面的语句，而只能获得一个生成器对象（generator object）。这个对象包含了函数的原始代码和函数调用的状态，其中函数状态包括函数中变量值以及当前的执行点。  
    
    ***函数遇见yield语句后会暂停，返回当前的值并储存函数的调用状态。当再次调用`_next__()`，从函数上次停止的状态继续执行，直到再次遇见yield语句***。但一般不直接调用`__next__()`方法，而是用for循环进行迭代
  ```python
  def countdown(n):
      print("count down!")
      while n > 0:
          yield n   # 函数暂停、返回当前值、存储函数调用状态
          n -= 1
          
  # 用__next__()方法
  gen = countdown(5); gen   # 输出<generator object countdown at 0x.......>
  gen.__next__()  
  gen.__next__()    

  # 用for循环
  for i in countdown(5):
      print(i)
  ```

* 和列表之间的转换
  ```python
  # 列表
  [ x ** 3 for x in range(5)]
  >>> [0, 1, 8, 27, 64]

  # 生成器
  (x ** 3 for x in range(5))
  >>>  <generator object <genexpr> at 0x000000000315F678>

  # 两者之间转换
  list( (x ** 3 for x in range(5)) )
  ```

### 3.2.2 迭代器与可迭代对象(Iterable vs. Iterator)
参见： https://www.cnblogs.com/wj-1314/p/8490822.html
* 可直接作用于for循环的对象统称为可迭代对象 --- Iterable  
含有`__next__()`方法的对象都是一个迭代器，所以上文说的**生成器是一种特殊的迭代器** --- Iterator
* list， tuple， dict， str等都是Iterable对象， 但不是Iterator
* 可用`isinstance()`可判断对象是否Iterable或是否是Iterator
```python
from collections import Iterable
from collections import Iterator

isinstance((x for x in range(10)), Iterable)
isinstance([], Iterable)

isinstance((x for x in range(10)), Iterator)
isinstance([], Iterator)
```
* 可用iter()函数把list、dict、str等Iterable对象变成Iterator
```python
isinstance(iter([]), Iterator)
isinstance(iter('abc'), Iterator)
```

<br>

## 3.3 parser
test.py文件的书写：
```python
#!/usr/bin/env python3
import argparse

if __name__ = "__main__":
    parser = argparse
    parser.add_argument("--model_dir", type=str, default="data/model_epoch46.chkpt",
        help="directory to model")
    parser.add_argument("--dataset_dir", type=str, default="/data/test.p",
        help="directory to dataset?")     

    args = parser.parse_args()

    model_dir = args.model_dir   
    dataset_dir = args.dataset_dir
```
命令行中用法： `python3 test.py --model_dir=data/model_epoch01.chkpt`  

<br>

## 3.4 变量作用域
* 每个变量都有作用域，Python中除了`def/class/lambda`外，其他如`if/elif/else/， try/except， for/while`等都不能改变变量作用域  
* 因为函数(def)会改变变量作用域，在函数内部定义的变量，是局部变量，函数外外面看不到。***局部变量会和同名全局变量冲突***
* 带`self.`的 类的成员变量作用域只在类内部，***不会与同名全局变量/局部变量都冲突***
    ```python
    class test:
        def __init__(self):
            self.global_var = 3;    # 类的成员变量不会和同名全局变量冲突
        def main(self):
            local_var = 2           # 类的成员变量不会和同名局部变量冲突
            self.local_var = 4;    
            print(global_var, local_var, self.global_var, self.local_var)
            global global_var       # 提示warning，提醒用户全局变量可能被修改。没有global关键词会报错
            global_var = 5          # 在函数中修改全局变量

    if __name__ == '__main__':
        global_var = 1              # if不改变作用域，所以是全局变量
        test().main()               # 或 s = test(); s.main() 但test.main()会报错，因为并没有先初始化类
        
    print(global_var)               # 此处v是全局变量
    # print(local_var) 会报错

    # 输出结果分别为1 2 3 4 5
    ```
* global和nonlocal关键字的使用: https://blog.csdn.net/youngbit007/article/details/64905070
  * python变量查找法则：LEGB（见闭包一节）。  
  * global关键字用于在函数或其他局部作用域中使用全局变量。但是如果不修改全局变量也可以不使用global关键字，直接使用即可（例如print，例子见上面)。  
  * nonlocal关键字用来在函数或其他作用域中使用外层 **(非全局)** 变量
    ```python
    def funx():
        x=5
        def funy():
            nonlocal x
            x += 1
            return x
        return funy

    if __name__ == "__main__":
        a=funx()
        print(a())
    ```

<br>

## 3.5 闭包，装饰器，语法糖
参见 https://www.zhihu.com/question/25950466/answer/31731502
### 3.5.1 闭包
> 所谓闭包，就是将组成函数的语句和这些语句的执行环境打包在一起时，得到的对象
```python
#foo.py
filename = "foo.py"

def call_fun(f):
    return f()
```
```python
# func.py
import foo

filename = "func.py"
def show_filename():
    return "filename: %s" % filemame

if __name__ == "__main__":
    print(foo.call_fun(show_filename))  # 返回 filename:func.py
```
这个例子说明了尽管 show_filename 函数的调用发生在 foo.py 文件内，但 show_filename 函数包含了其执行所需的整个环境（也即filename变量的值），构成了一个闭包，其中 filename 变量的查找遵循`Local-> Enclosing-> Global-> Builtin`顺序查找:  
* 本地函数(show_filename内部)：通过任何方式赋值的，而且没有被global关键字声明为全局变量的filename变量
* 直接外围空间(show_filename外部)：如果有多层嵌套，则由内而外逐层查找，直至最外层的函数
* 全局空间(func.py和其import部分)
* 内置模块(\_\_builtin\_\_)  

闭包在其捕捉的执行环境(def语句块所在上下文)中，也遵循LEGB规则逐层查找，直至找到符合要求的变量，或者抛出异常。  
这说明函数内可以查找到函数外定义的变量，这是显而易见的，例如全局变量理所当然可以在任何位置被使用。***这与前面讲的变量作用域不冲突，就变量本身而言，函数内定义的变量生命周期只在函数内部。但函数被作为闭包返回，为函数内变量续了命***

### 3.5.2 装饰器
> 装饰器本质上是一个Python 函数或类。运用闭包能封存上下文的特性，它可以让其他函数或类在不做任何代码修改的前提下增加额外功能，装饰器的返回值也是一个函数/类对象

下面的例子，用闭包的性质为add函数添加了新的功能：
```python
def checkParams(fn):
    def wrapper(a, b):
        if isinstance(a, (int, float)) and isinstance(b, (int, float)):
            return fn(a,b)  # 解释器按照LEGB法则找到fn，也即add函数对象的一个引用
        print("type incompatible")
        return
    return wrapper

def add(a, b):
    return a+b

if __name__ == "__main__":
    add = checkParams(add)  # 返回的是一个wrapper这个函数的闭包
    add(3, "hello")
```

### 3.5.3 装饰器的语法糖
> 装饰器的语法糖：在写法上简化上面的代码，参见：https://www.jianshu.com/p/fd746acbdf1e
```python
def checkParams(fn):
    # 这个函数的实现同上，不变

@checkParams
def add(a, b):
    return a+b

if __name__ == "__main__":
    add(3, "hello")
```
简单来说，就是将`@checkParams`写在`add(a,b)`定义上面，等效于`add = checkParams(add)`

### 3.5.4 用类写一个多重的，带参数的装饰器
```python
class add_prefix(object):
    def __init__(self, word):
        self.word = word
    def __call__(self, fn):
        def wrapper(*args, **kwargs):
            return self.word + fn(*args, **kwargs)
        return wrapper

@add_prefix("dear ")
@add_prefix("Prof.")
def hello(name):
    return name

if __name__ == "__main__":
    print(hello("Chen"))
```
上面的两个装饰器等价于:
```
def hello(name):
    return name
    
if __name__ == "__main__":
    hello = add_prefix("dear ")( add_prefix("Prof.")(hello) )
    print(hello("Chen"))
```

### 3.5.5 其他
[函数内部的变量在函数执行完后就销毁，为什么可变对象却能保存上次调用时的结果呢？](https://www.zhihu.com/question/264533969)



