## 一、基础

### 1、安装

[Python 官网](https://www.python.org/)

[PyCharm Community](https://www.jetbrains.com/edu-products/download/download-thanks-pce.html)

> Hello World

```python
# 打印
print("Hello World!")


# 运行
> python ./hello.py
```

### 2、基础

#### （1）注释

```python
# 单行注释

"""
多行注释 1
多行注释 2
"""
```

#### （2）变量

```python
# python 中变量没有类型，存储的值有类型
name = "ycz"
# <class 'str'>
print(type(name))

name = 18
# <class 'int'>
print(type(name))

# 数据类型转换
# <class 'str'>
print(str(name), type(str(name)))
```

#### （3）运算符

```
# 算术运算
+、-、*、/、%

# 取整除 //
9//2 => 4

# 指数
2**10 => 1024

# 赋值运算
=、+=、-=、*=、/=、%=、**=、//=

# 比较运算符
==、!=、>、<、>=、<=
```

#### （4）input

```python
# input 接收控制台输入
name = input("姓名：")
print("欢迎：", name)
```

### 3、数据类型

```
数字型（number）
    整型（int）
    浮点型（float）
    复数（complex）
    布尔（Flase|True）

数据容器：
    字符串（str）：字符元组
    列表（list）：有序的可变序列
    元组（tuple）：有许多不可变序列
    集合（set）：无序不重复集合
    字典（dict）：无序 k-v 集合


# 查看数据类型
type() 
```

#### （1）str

```python
# 定义字符串
name = 'ycz'
name = "ycz"

# 定义多行字符串
name = """
    hello
    ycz
    18
"""

# 拼接
welcome = "Welcome"
name = "ycz"
age = 18

print(welcome + " " + name)

# 格式化 %s
hello = "%s %s, age: %d" % (welcome, name, age)
print(hello)

# 格式化 f"{...}"
print(f"My name is {name}, I'm {age} years old")
```

#### （2）list

```python
# 定义两个空列表
arr1 = list()
arr2 = []

# [] []
print(arr1, arr2)
# <class 'list'>
print(type(arr1), type(arr2))

arr3 = [1, 2, 3]
# 列表的元素类型可以不同
arr4 = ['a', 2, "str"]

print(arr3, arr4)

# 下标索引：从左向右
print(arr3[0])  # 1
# 下标索引：从右向左
print(arr4[-1])  # str

"""
  list 方法
"""
# 返回元素下标，未找到时出错
list_name.index(元素)

# 往 list 中指定下标插入元素
list_name.insert(idx, 元素)

# 尾部追加一个
list_name.append(元素)

# 尾部追加一批
list_name.extend(sublist)

# 删除指定下标元素
del(list_name[idx])    # 无返回值
v = list_name.pop(idx) # 移除元素，返回被移除元素
list_name.remove(元素)  # 从左向右删除指定元素（仅删除一次）

# 清空列表
list_name.clear()

# 统计元素数量
list_name.count(元素)

# 列表总长度
len(list_name)
```

#### （3）tuple

```python
# 定义空元组
tu = ()
tu = tuple()

tu = (1, 'ab', True)

# 定义一个元素时须在后面加个逗号 ','
# 否则变成 str
tu = ("abc", )

# 注：tuple 与 list 特性相似，唯一的不同是 内容不可修改
# 特例：tuple 中 嵌套的 list 成员可以修改

# 元组的方法，同上
# index()、count()、len()
```

#### （4）序列切片

> 序列：str、list、tuple

```python
# 步长为正时，从左向右取
# 步长为负时，从右向左取
subList = 序列[起始下标:结束下标:步长]
```

#### （5）set

```python
# 空集合
set1 = set()

# 集合方法
set1.add(member)
set1.remote(member)
set1.pop()
set1.clear()
set1.difference(set2)
set1.difference_update(set2)
set1.union(set2)
len(set1)
```

#### （6）数据容器的通用操作

> 数据容器：str、list、tuple、set

```python
#  的通用操作

# 通用遍历 
for x in ... :

# 最大值
max()

# 最小值
min()

# 长度
len()

# 转为 list
list()

# 转换为元组
tuple()

# 转换为字符串
str()

# 转换为集合
set()

# 排序
sorted(序列, [reverse=True])
```

#### （7）dict

```python
dicts = {}
dicts = dict()

dicts = {key: value, key: value, ...}

dicts.pop(key)
dicts.clear()
dicts.keys()
dicts.values()
len(dicts)
```



### 4、流程控制

#### （1）if

```python
age = 10

if age >= 18:
    print("大人")
elif age >= 10:
    print("儿童")
else:
    print("小孩")
```

#### （2）while

```python
num = 10

while num > 0:
    print(num)
    num -= 1
```

#### （3）for

```python
letter = "abcdefghijklmnopqrstuvwxyz"

for byte in letter:
    print(byte)
```

#### （4）range

```python
# range(n)
# 获取一个从 0 ~ n-1 的数字序列

# range(m, n)
# 获取一个从 m ~ n-1 数字序列

# range(m, n, step)
# 获取一个从 m ~ n-1 数字序列，步长为 step（默认：1）

for i in range(1, 10, 2):
    print(i)
```

#### （5）continue & break

```python
for i in range(1, 10, 2):
    if i == 5:
        continue

    print(i)
    

for i in range(1, 10, 2):
    if i == 5:
        break

    print(i)
```

### 5、函数

#### （1）定义

```python
# 默认值参数必须放在最后
def add(x, y, flag=True):
    """
    函数说明：带默认参数的函数
    :param x:
    :param y:
    :param flag: True 时：a + b，False 时：a - b
    :return: x ± y, ±
    """

    if flag:
        return x + y, "+"
    else:
        return x - y, "-"


# 使用默认参数 +
print("x + y = ", add(5, 6))
print("x + y = ", add(5, y=6))
# 按参数名传参，位置可以打乱
# 按位置传参和按参数名传参混用时，按位置传的参数必须在前
print("x - y = ", add(5, flag=False, y=6))
```

#### （2）返回值 None

```python
# 无返回值的函数，返回类型为 None
def hello():
    print("Hello")
    # return None

# None
print("返回值：", hello())
# <class 'NoneType'>
print("返回值类型：", type(hello()))
# None == False
print("None == False: ", hello() == False)
```

#### （3）变量作用域

```python
age = 18

def test():
    # 局部变量
    name = "ycz"
    
    # 全局变量
    global age
    age = 20

test()
# 20
print(age)
```

#### （4）不定参数

```python
def info(*args):
    # <class 'tuple'> (1, 'a', 'Dd')
    print(type(args), args)
    
info(1, 'a', "Dd")
```

#### （5）匿名函数

```python
lambda params, params, ...: 函数体（只能写一行代码）

func_add = lambda x, y: x + y
print(func_add(1, 2))
```

#### （6）闭包

```python
"""
优点：
 - 无需定义全局变量即可实现通过函数，持续的访问、修改某个值
 - 闭包使用变量的作用域在函数内，不会被篡改

缺点：
 - 导致内存逃逸，所引用的变量内存空间不被释放，一直占用内存
"""

def crs(balance=0):
    """
    存取款一体机
    :param money: 金额
    :param deposit: True 为存款，False 为取款，默认：取款
    :return:
    """

    def inner(money: int, deposit=False):
        # 在闭包函数中用 nonlocal 声明要修改的外层函数的变量
        nonlocal balance

        if deposit:
            balance += money
            print(f"您的存款金额为：{money}\t【账户余额：{balance}】")
        else:
            if money > balance:
                print("余额不足")
                return

            balance -= money
            print(f"您的取款金额为：{money}\t【账户余额：{balance}】")

    return inner


crs = crs()
crs(100, True)
crs(200, True)
crs(300, True)
crs(100)
```

#### （7）装饰器

> 装饰器：在不破坏被装饰对象源代码和调用方式的前提下，为被装饰对象添加额外的功能。
>
> 被装饰对象可以是：函数、方法、类。
>
> 装饰器应用场景：插入日志、性能测试、事务处理、缓存、权限校验。

- 装饰器底层实现（闭包方式）

```python
# 1 使用闭包形式实现简易装饰器
import time


def decorator(func):
    """
    装饰器：打印函数的执行时间
    """

    def wrapper(*args, **kwargs):
        start = time.time()
        # *args, **kwargs 解决装饰器传参问题
        res = func(*args, **kwargs)
        end = time.time()
        print("function run duration: %.2f s" % (end - start))

        return res

    return wrapper


def factorial(n: int):
    time.sleep(1)

    if n == 0:
        return 1
    elif n > 0:
        return n * factorial(n - 1)
    else:
        raise ValueError("输入值不在预期范围内")

"""
无装饰器
  1! =  1
"""
print("1! = ", factorial(1))

"""
装饰器模式下：
  function run duration: 3.00 s
  2! =  2
"""
# 匿名函数 lambda: factorial(2) ，传递阶乘参数
fn = decorator(lambda: factorial(2))
print("2! = ", fn())
```

- 装饰器语法糖：@<装饰器函数>

```python
import time


def decorator(func):
    """
    装饰器：打印函数的执行时间
    """

    def wrapper(*args, **kwargs):
        start = time.time()
        # *args, **kwargs 解决装饰器传参问题
        res = func(*args, **kwargs)
        end = time.time()
        print("function run duration: %.2f s" % (end - start))

        return res

    return wrapper


@decorator
def factorial(n: int):
    time.sleep(1)

    if n == 0:
        return 1
    elif n > 0:
        return n * factorial(n - 1)
    else:
        raise ValueError("输入值不在预期范围内")


"""
  装饰器模式下：
    # 每次递归都执行了装饰器
    function run duration: 1.00 s 
    function run duration: 2.00 s
    function run duration: 3.00 s
    function run duration: 4.00 s
    3! =  6
"""
print("3! = ", factorial(3))
```

### 6、异常处理

```python
try:
    arr = [1, 2, 3]
    print(arr[3])
except Exception as e:
    print("捕获异常：", e)
else:
    print("【else 可选】 无异常")
finally:
    print("【finally 可选】 有无异常都会执行")
    
print("正常运行")
```

### 7、包与模块

#### （1）模块

```python
# vim hello.py

def hello(name):
    print(f"hello {name}")

# 其它包 import hello 时，默认会执行 hello("ycz")
# 加入 __main__ 变量后，其它包导入不会执行
# 用于编写单元测试
if __name__ == '__main__':
    hello("ycz")
```

#### （2）包

```
pkg/              # pkg 包
  ├─ __init__.py  # 【必须】存在该文件时，才叫 python 包，否则是普通文件夹
  ├─ hello.py     # hello 模块，包由一个个模块组成，模块名同文件名
  ├─ test.py      # test 模块
```

> `__init__.py`

```python
# 一般为空文件
# 但必须存在

# 当使用 from hello import * 导入时，
# 只能导入 __all__ 变量列表中指定的模块或方法
# 其它导入方式不受限制，如 import hello | from hell import internal
__all__ = []
```

> `hello.py`

```python
# 当使用 from hello import * 导入时，
# 只能导入 __all__ 变量列表中指定的方法
# 其它导入方式不受限制，如 import hello | from hell import internal
__all__ = ["say"]


def say(name):
    print(f"hello {name}")


def internal(name):
    print(f"internal {name}")


# 本包单元测试用
# 其他包引用时，不执行该方法
if __name__ == '__main__':
    say("ycz")
    internal("in")
```

> `test.py`

```python
import hello


hello.say("test_ycz")
hello.internal("in")

# ============================

from hello import *

say("external")
# internal(external) # error
```

### 8、类型注解

> 类型注解用于编辑器提示
>
> 提示性而非强制性，即使注解错误也可正常运行

```python
# =============
#  变量的类型注解
# =============

# 语法 1：  变量: 类型
my_dict: dict = {"name": "ycz", "age", 18}

# 语法 2：  在注释中，# type: 类型
name = "ycz"  # type: str
age = 18      # type: int

# =============
#  函数的类型注解
#   - 形参注解
#   - 返回值注解
# =============
def 函数方法名(var: Type> [| Type | ...], var: Type [| Type | ...], ...) -> Type [| Type | ...]:
    pass

def add(x: int | float, y: int) -> int | float:
    return x + y

# ==============
#  Union 类型注解
# ==============
from typing import Union

Union[Type, Type, ...]

my_dict: dict[str, Union[str, int]] = {"name": "ycz", "age", 18}

def add(x: Union[int, float], y: int) -> Union[int, float]:
    return x + y
```



## 二、面向对象

```python
class Student0:
    Name = None

    def say(self):
        print(f"hello student {self.Name}")


stu = Student0()
stu.Name = "ycz"
stu.say()

class Student:
    name = None
    sex = None
    age = None
    score = None
    # __ 开头的成员变量，私有成员变量
    __private = None

    # 构造函数
    def __init__(self, name, sex, age, score, private):
        self.name = name
        self.sex = sex
        self.age = age
        self.score = score
        self.__private = private

    # 成员方法
    def say(self):
        print(f"hello student {self.name}")

    # 私有成员方法
    def __internal(self):
        print("internal func", self.__private)

    # 私有成员访问入口
    def index(self):
        self.__internal()

    # ===== 魔术方法 ===== #

    # 用于实现类对象转字符串
    def __str__(self):
        print(f"name: {self.name}, sex: {self.sex}, age: {self.age}, __private: {self.__private}")

    # 用于两个对象进行 >、< 的比较运算
    def __lt__(self, other):
        return self.score < other.score

    # 用于两个对象进行 >= <= 的比较
    def __le__(self, other):
        return self.score <= other.score

    # 用于两个对象进行 == 的比较
    def __eq__(self, other):
        return self.score == other.score


stu1 = Student("ycz", 'man', 18, 99.9, True)
stu1.__str__()

stu2 = Student("yyy", 'woman', 17, 88.8, False)
stu2.__str__()

# AttributeError: 'Student' object has no attribute '__private'
# print(stu1.__private)

# AttributeError: 'Student' object has no attribute '__internal'
# stu1.__internal()

stu1.index()
stu2.index()

print(stu1 < stu2)
print(stu1 > stu2)

print(stu1 <= stu2)
print(stu1 >= stu2)

print(stu1 == stu2)
```

### 1、继承

```python
# 多继承优先级：Parent1 > Parent2 > ...
class Son(Parent1, Parent2, ...):
    # 当不需要写额外代码时，用 pass
    pass

    # 通过定义重名成员和方法覆盖父类的成员和方法
    # 通过父类名或 super() 调用父类成员和方法，
    #   Parent1.member、Parent1.func()
    #   super().member、super().func()
```

### 2、多态

```python
# 抽象类（接口）
class Animal:
    # 抽象方法：没有实现的方法
    def speak(self):
        pass


class Dog(Animal):
    def speak(self):
        print("汪汪汪")


class Cat(Animal):
    def speak(self):
        print("喵喵喵")


def call(animal: Animal):
    animal.speak()


dog = Dog()
cat = Cat()

call(dog)
call(cat)
```

## 三、多线程编程

> Python 的多线程可以通过 threading 模块来实现。

```python
import threading

thread = threading.Thread([group [, target [, name [, args [, kwargs]]]]])

- group:  为将来实现 ThreadGroup 类预留
- target: run() 方法要调用的可调用对象。默认为None，表示不调用任何内容。
- name:   线程名称。默认情况下，一个唯一的名称以“Thread-N”的形式构建，其中 N 是一个小的十进制数字。
- args:   目标调用的参数列表或元组。默认为 ()。
- kwargs: 目标调用的关键字参数字典。默认为 {}。

    
# 启动线程，让线程开始工作
thread.start()
```

> 案例

```python
import threading
import time


def sing(song):
    while True:
        print(f"我正在唱：《{song}》")
        time.sleep(2)


def dance(name):
    while True:
        print(f"我正在跳：《{name}》")
        time.sleep(2)


if __name__ == "__main__":
    # sing 没结束，无法 dance
    # sing("生日快乐")
    # dance("芭蕾")

    # 创建 2 个线程
    sing_thread = threading.Thread(target=sing, args=["生日快乐"])
    dance_thread = threading.Thread(target=dance, kwargs={"name": "芭蕾"})

    # 启动线程
    sing_thread.start()
    dance_thread.start()
```



## 四、标准库

### 1、文件操作

#### （1）Read

> open

```python
# 以 r 只读模式打开当前文件
f = open(__file__, "r", encoding="UTF-8")

# 1 用 f.read() 读取文件全部内容
print(f.read())

# 2 用 f.readline() 读取一行
# print(f.readline())

# 3 用 f.readlines() 读取全部行
# print(f.readlines())

# 4 遍历全部行
# for line in f.readlines():
#     print(line)
# 等价
# for line in f:
#     print(line)

# 关闭文件
f.close()
```

> with open 语法

```python
# with open 会在语句块结束后，自动调用 f.close()
with open(__file__, "r", encoding="UTF-8") as f:
    print(f.read())
```

#### （2）Write

```python
# 以 w 截断写入模式打开当前文件
f = open(__file__, "w", encoding="UTF-8")

# 1 用 f.write() 写入内容
f.write("print('文件已被重置...')")

# 2 用 f.flush() 将内容从内存写入到磁盘
# f.flush()

# 3 关闭文件，
# 内置 f.flush() 将文件落盘后再关闭
f.close()
```

```python
# 以 w 截断写入模式打开当前文件
f = open(__file__, "a", encoding="UTF-8")

# 1 用 f.write() 写入内容
f.write("\nprint('向文件末尾追加一行...')")

# 2 用 f.flush() 将内容从内存写入到磁盘
# f.flush()

# 3 关闭文件，
# 内置 f.flush() 将文件落盘后再关闭
f.close()
```



