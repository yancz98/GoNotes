[Python 官网](https://www.python.org/)

[Python 编辑器 PyCharm Community](https://www.jetbrains.com/edu-products/download/download-thanks-pce.html)



Hello World

```python
# 打印
print("Hello World!")


# 运行
> python ./hello.py
```

数据类型

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
 
# 定义一个字符串
name = 'ycz'
name = "ycz"
name = """ycz"""

# 查看数据类型
type()  
```

注释

```python
# 单行注释

"""
多行注释 1
多行注释 2
"""
```

变量

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

运算符

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

字符串

```python
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

input

```
name = input("姓名：")
print("欢迎：", name)
```

if

```python
age = 10

if age >= 18:
    print("大人")
elif age >= 10:
    print("儿童")
else:
    print("小孩")
```

while

```python
num = 10

while num > 0:
    print(num)
    num -= 1
```

for

```python
letter = "abcdefghijklmnopqrstuvwxyz"

for byte in letter:
    print(byte)
```

range

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

continue & break

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

func

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


"""
  函数返回值 None 类型
"""  

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

"""
  函数中变量的作用域
"""

age = 18

def test():
    # 局部变量
    name = "ycz"
    
    # 全局变量
    global age
    age = 20

test()
print(age)


"""
  不定长参数
"""
def info(*args):
    # <class 'tuple'> (1, 'a', 'Dd')
    print(type(args), args)
    
info(1, 'a', "Dd")


"""
  匿名函数
"""
lambda params, params, ...: 函数体（只能写一行代码）

func_add = lambda x, y: x + y
print(func_add(1, 2))
```

list

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

tuple

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

序列切片

```python
# 序列可以是 str、list、tuple
# 步长为正时，从左向右取
# 步长为负时，从右向左取
subList = 序列[起始下标:结束下标:步长]
```

set

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

dict

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

数据容器的通用操作

```python
# list、tuple、str、set 的通用操作

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

异常处理

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

自定义模块

```python
# vim hello.py

def hello(name):
    print(f"hello {name}")

# 其它包 import hello 时，默认会执行 hello("ycz")
# 加入 __main__ 变量后，其它包导入不会执行
if __name__ == '__main__':
    hello("ycz")
```

包与模块

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

面向对象

```python
class Student:
    Name = None

    def say(self):
        print(f"hello student {self.Name}")


stu = Student()
stu.Name = "ycz"
stu.say()
```

继承

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

