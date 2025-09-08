## 一、基础

### 1、安装

[Python 官网](https://www.python.org/)

[PyCharm Community](https://www.jetbrains.com/edu-products/download/download-thanks-pce.html)

> Hello World

```python
$ vim hello.py

# 打印
print("Hello World!")


# 运行
$ python hello.py
Hello World!
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

# 作用域
# 只有模块（module），类（class）以及函数（def、lambda）才会引入新的作用域，其它的代码块（如 if/elif/else/、try/except、for/while等）是不会引入新的作用域的


# 使用 global 可以在函数中修改全局变量
a = 10
def test():
    global a
    a = a + 1
    print(a) # 11
test()

# 使用 nonlocal 可以在嵌套（闭包）函数中修改外部函数的变量。
def outer():
    num = 10
    def inner():
        nonlocal num   # nonlocal关键字声明
        num = 100
        print(num) # 100
    inner()
    print(num) # 100
outer()
```

#### （3）运算符

```
# 算术运算
+、-、*、/、//、%、**

# 整除 //，但返回值不一定为 int，跟分子分母类型有关
>>> 9 // 2; 9.0 // 2; 9 // 2.0
4
4.0
4.0


# 指数 2^10
>>> 2**10
1024

# 赋值运算
=、+=、-=、*=、/=、%=、**=、//=

# 比较运算符
==、!=、>、<、>=、<=

# 位运算符
&、|、^、~、<<、>>

# 逻辑运算符
and、or、not

# 海象运算符（:=）
# 在表达式中同时进行赋值和返回赋值的值。
# Python3.8 版本新增运算符。
if (n := 10) > 5:
    print(n)
    
if (n := len(a)) > 10:
    print(f"List is too long ({n} elements, expected <= 10)")
    
# 成员运算符
in、not in

# 如果 x 在序列中返回 True
if x in sequence:
    pass
    
# 如果 y 不在序列中返回 True
if y not in sequence:
    pass
    
# 身份运算符
is、not is

# is 是判断两个标识符是不是引用自一个对象
if x is y: # 类似 id(x) == id(y)
    pass

# is 与 == 的区别：
# is 用于判断两个变量引用对象是否为同一个，
# == 用于判断引用变量的值是否相等。
```

#### （4）input

```python
# input 接收控制台输入
name = input("姓名：")
print("欢迎：", name)
```

#### （5）Python 关键字

```python
# 标准库 keyword 模块，可以输出当前版本的所有关键字：
>>> import keyword
>>> keyword.kwlist
['False', 'None', 'True', 'and', 'as', 'assert', 'async', 'await', 'break', 'class', 'continue', 'def', 'del', 'elif', 'else', 'except', 'finally', 'for', 'from', 'global', 'if', 'import', 'in', 'is', 'lambda', 'nonlocal', 'not', 'or', 'pass', 'raise', 'return', 'try', 'while', 'with', 'yield']
```



| **类别**     | **关键字** | **说明**                               |
| :----------- | :--------- | :------------------------------------- |
| **逻辑值**   | `True`     | 布尔真值                               |
|              | `False`    | 布尔假值                               |
|              | `None`     | 表示空值或无值                         |
| **逻辑运算** | `and`      | 逻辑与运算                             |
|              | `or`       | 逻辑或运算                             |
|              | `not`      | 逻辑非运算                             |
| **条件控制** | `if`       | 条件判断语句                           |
|              | `elif`     | 否则如果（else if 的缩写）             |
|              | `else`     | 否则分支                               |
| **循环控制** | `for`      | 迭代循环                               |
|              | `while`    | 条件循环                               |
|              | `break`    | 跳出循环                               |
|              | `continue` | 跳过当前循环的剩余部分，进入下一次迭代 |
| **异常处理** | `try`      | 尝试执行代码块                         |
|              | `except`   | 捕获异常                               |
|              | `finally`  | 无论是否发生异常都会执行的代码块       |
|              | `raise`    | 抛出异常                               |
| **函数定义** | `def`      | 定义函数                               |
|              | `return`   | 从函数返回值                           |
|              | `lambda`   | 创建匿名函数                           |
| **类与对象** | `class`    | 定义类                                 |
|              | `del`      | 删除对象引用                           |
| **模块导入** | `import`   | 导入模块                               |
|              | `from`     | 从模块导入特定部分                     |
|              | `as`       | 为导入的模块或对象创建别名             |
| **作用域**   | `global`   | 声明全局变量                           |
|              | `nonlocal` | 声明非局部变量（用于嵌套函数）         |
| **异步编程** | `async`    | 声明异步函数                           |
|              | `await`    | 等待异步操作完成                       |
| **其他**     | `assert`   | 断言，用于测试条件是否为真             |
|              | `in`       | 检查成员关系                           |
|              | `is`       | 检查对象身份（是否是同一个对象）       |
|              | `pass`     | 空语句，用于占位                       |
|              | `with`     | 上下文管理器，用于资源管理             |
|              | `yield`    | 从生成器函数返回值                     |

### 3、数据类型

```python
# 查询变量所指的对象类型 type()
>>> a = 10
>>> type(a) 
<class 'int'>

>>> a = 10.0
>>> type(a)
<class 'float'>

# 是否为对象实例 isinstance()
>>> isinstance(a, float)
True

# type() 和 isinstance() 的区别
# type() 不会认为子类是一种父类类型。
# isinstance() 会认为子类是一种父类类型。

# Python3 中，bool 是 int 的子类
>>> type(True) == int
False
>>> isinstance(True, int)  
True

# 是否为对象子类 issubclass()
>>> issubclass(bool, int) 
True


# 删除单个或多个对象 del
del var1[, var2[, var3[...., varN]]]
```

#### 3.1、数字（number）

```
数字型（number）
    整型（int）
        布尔（Flase|True） # bool 是 int 的子类
    浮点型（float）
    复数（complex）
    decimal
    fraction
```

> 数学函数
>
> import math

| 函数            | 返回值 ( 描述 )                                              |
| :-------------- | :----------------------------------------------------------- |
| abs(x)          | 返回数字的绝对值，如abs(-10) 返回 10                         |
| ceil(x)         | 返回数字的上入整数，如math.ceil(4.1) 返回 5                  |
| cmp(x, y)       | 如果 x < y 返回 -1, 如果 x == y 返回 0, 如果 x > y 返回 1。 **Python 3 已废弃，使用 (x>y)-(x<y) 替换**。 |
| exp(x)          | 返回e的x次幂(ex),如math.exp(1) 返回2.718281828459045         |
| fabs(x)         | 以浮点数形式返回数字的绝对值，如math.fabs(-10) 返回10.0      |
| floor(x)        | 返回数字的下舍整数，如math.floor(4.9)返回 4                  |
| log(x)          | 如math.log(math.e)返回1.0,math.log(100,10)返回2.0            |
| log10(x)        | 返回以10为基数的x的对数，如math.log10(100)返回 2.0           |
| max(x1, x2,...) | 返回给定参数的最大值，参数可以为序列。                       |
| min(x1, x2,...) | 返回给定参数的最小值，参数可以为序列。                       |
| modf(x)         | 返回x的整数部分与小数部分，两部分的数值符号与x相同，整数部分以浮点型表示。 |
| pow(x, y)       | x**y 运算后的值。                                            |
| round(x [,n\])  | 返回浮点数 x 的四舍五入值，如给出 n 值，则代表舍入到小数点后的位数。**其实准确的说是保留值将保留到离上一位更近的一端。** |
| sqrt(x)         | 返回数字x的平方根。                                          |

> 随机数函数
>
> import random

| 函数                               | 描述                                                         |
| :--------------------------------- | :----------------------------------------------------------- |
| choice(seq)                        | 从序列的元素中随机挑选一个元素，比如random.choice(range(10))，从0到9中随机挑选一个整数。 |
| randrange ([start,\] stop [,step]) | 从指定范围内，按指定基数递增的集合中获取一个随机数，基数默认值为 1 |
| random()                           | 随机生成下一个实数，它在[0,1)范围内。                        |
| seed([x\])                         | 改变随机数生成器的种子seed。如果你不了解其原理，你不必特别去设定seed，Python会帮你选择seed。 |
| shuffle(lst)                       | 将序列的所有元素随机排序                                     |
| uniform(x, y)                      | 随机生成下一个实数，它在[x,y]范围内。                        |

#### 3.2、序列（sequence）

- 不可变序列：
  - 字符串（str）：由代表 Unicode 码位的值组成的序列。
  - 元组（tuple）：元组中的条目可以是任意 Python 对象。
  - 字节串（bytes）：条目是 8 比特位的字节，以取值范围 0 <= x < 256 内的整数表示。
- 可变序列：
  - 列表（list）：条目可以是任意 Python 对象。列表由用方括号括起并由逗号分隔的多个表达式构成。
  - 字节数组。        

#### （1）字符串（str）

> 字符串是由 Unicode 码位构成的不可变序列。

```python
# 定义字符串（单引号，双引号都可以）
name = 'ycz'
name = "ycz"

# 定义多行字符串 """..."""
print("""
    hello
    ycz
    18
""")

# 忽略开头的换行符 \
print("""\
    hello
    ycz
    18
""")

# 拼接 +
>>> 'hello ' + 'ycz'
'hello ycz'

# 重复 *
>>> 'Ha' + 2 * 'p' + 'y'
'Happy'

# 相邻的多个字符串字面值会自动合并（用于拼接分隔开的长字符串）
>>> text = ('Put several strings within parentheses '
... 'to have them joined together.')
>>> text
'Put several strings within parentheses to have them joined together.'


# 格式化 %s
hello = "%s %s, age: %d" % (welcome, name, age)
print(hello)

# 格式化 f"{...}"
print(f"My name is {name}, I'm {age} years old")

# 通过下标访问字符串
>>> language = 'Python'
>>> language[0]
'P'
>>> language[-1] # 最后一个元素
'n'
>>> language[0:2] # [0,2)
'Py'
>>> language[10] # 索引越界会报错
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
IndexError: string index out of range
>>> language[10:] # 但是，切片会自动处理越界索引
''

# 输出原始字符串（不转义） r
>>> print('C:\some\name')
C:\some
ame
>>> print(r'C:\some\name')
C:\some\name
```

> 添加进度条图形和百分比

```python
import time

for i in range(101): 
    bar = '[' + '=' * (i // 2) + ' ' * (50 - i // 2) + ']'
    print(f"\r{bar} {i:3}%", end='', flush=True)
    time.sleep(0.05)
else:
    print()
```

> 格式化符号

| 符  号 | 描述                                 |
| :----- | :----------------------------------- |
| %c     | 格式化字符及其ASCII码                |
| %s     | 格式化字符串                         |
| %d     | 格式化整数                           |
| %u     | 格式化无符号整型                     |
| %o     | 格式化无符号八进制数                 |
| %x     | 格式化无符号十六进制数               |
| %X     | 格式化无符号十六进制数（大写）       |
| %f     | 格式化浮点数字，可指定小数点后的精度 |
| %e     | 用科学计数法格式化浮点数             |
| %E     | 作用同%e，用科学计数法格式化浮点数   |
| %g     | %f和%e的简写                         |
| %G     | %f 和 %E 的简写                      |
| %p     | 用十六进制数格式化变量的地址         |

> 格式化操作符辅助指令:

| 符号  | 功能                                                         |
| :---- | :----------------------------------------------------------- |
| *     | 定义宽度或者小数点精度                                       |
| -     | 用做左对齐                                                   |
| +     | 在正数前面显示加号( + )                                      |
| <sp>  | 在正数前面显示空格                                           |
| #     | 在八进制数前面显示零('0')，在十六进制前面显示'0x'或者'0X'(取决于用的是'x'还是'X') |
| 0     | 显示的数字前面填充'0'而不是默认的空格                        |
| %     | '%%'输出一个单一的'%'                                        |
| (var) | 映射变量(字典参数)                                           |
| m.n.  | m 是显示的最小总宽度,n 是小数点后的位数(如果可用的话)        |

```
# 首字母大写
'abc.capitalize()
'Abc'

# 
>>> "a".center(10, ' ')
'    a     '


```



#### （2）列表（list）

> 列表是有序的对象集合。列表中元素的类型可以不相同。列表是写在方括号 [] 之间，元素之间用逗号分隔。

```python
# 定义空列表的两种方式：
>>> arr1, arr2 = list(), []
>>> print(arr1, arr2)
[] []
>>> print(type(arr1), type(arr2))
<class 'list'> <class 'list'>

# 列表的元素类型可以不同
>>> arr1, arr2 = [1, 2, 3], ['a', 1.0, 2, "str", True]
# 拼接列表
>>> print(arr1 + arr2)
[1, 2, 3, 'a', 1.0, 2, 'str', True]
# 重复列表
>>> print(arr1 * 2)
[1, 2, 3, 1, 2, 3]

# 下标索引：正数从左向右，负数从右向左
>>> print(arr1[0], arr1[-1])  
1 3

# 切片
>>> print(arr2[1:3])
[1.0, 2]
>>> print(arr1[-1::-1]) # 反转 list
[3, 2, 1]


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

| 方法                                | 描述                                                         |
| :---------------------------------- | :----------------------------------------------------------- |
| list.append(obj)                    | 在列表末尾添加新的对象                                       |
| list.count(obj)                     | 统计某个元素在列表中出现的次数                               |
| list.extend(seq)                    | 在列表末尾一次性追加另一个序列中的多个值（用新列表扩展原来的列表） |
| list.index(obj)                     | 从列表中找出某个值第一个匹配项的索引位置                     |
| list.insert(index, obj)             | 将对象插入列表                                               |
| list.pop([index=-1\])               | 移除列表中的一个元素（默认最后一个元素），并且返回该元素的值 |
| list.remove(obj)                    | 移除列表中某个值的第一个匹配项                               |
| list.reverse()                      | 反向列表中元素                                               |
| list.sort( key=None, reverse=False) | 对原列表进行排序                                             |
| list.clear()                        | 清空列表                                                     |
| list.copy()                         | 复制列表                                                     |

> 列表推导式

```python
[表达式 for 变量 in 列表] 
或者 
[表达式 for 变量 in 列表 if 条件]

# 计算【1979 ~ 2025】的闰年
>>> [ y for y in range(1979, 2025) if (y % 4 == 0 and y % 100 != 0 or y % 400 == 0) ]
[1980, 1984, 1988, 1992, 1996, 2000, 2004, 2008, 2012, 2016, 2020, 2024]

# 过滤掉长度小于或等于3的字符串列表，并将剩下的转换成大写字母：
>>> names = ['Bob','Tom','alice','Jerry','Wendy','Smith']
>>> new_names = [name.upper() for name in names if len(name)>3]
>>> print(new_names)
['ALICE', 'JERRY', 'WENDY', 'SMITH']
```



#### （3）元组（tuple）

> 元组的元素不能修改。元组写在小括号 () 里，元素之间用逗号分隔。元组中的元素类型也可以不相同。
>
> 特例：tuple 中嵌套的 list 成员可以修改。

```python
# 定义空元组
>>> tu1, tu2 = tuple(), ()

# 定义一个元素时须在后面加个逗号 ','
# 区分它是一个元组而不是一个普通的值
tu = ("abc", )

tu = (1, 'ab', True)

# 元组的方法，同 list
```

> 元组推导式（生成器表达式）

```python
(expression for item in Sequence )
或
(expression for item in Sequence if conditional )

>>> a = (x for x in range(1,10))
>>> a
<generator object <genexpr> at 0x7faf6ee20a50>  # 返回的是生成器对象

>>> tuple(a)       # 使用 tuple() 函数，可以直接将生成器对象转换成元组
(1, 2, 3, 4, 5, 6, 7, 8, 9)
```



#### （4）序列的通过操作

> str、list、tuple 均适用一下方法：

| 运算                   | 结果：                                                       | 备注                                                         |
| :--------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| `x in s`               | 如果 *s* 中的某项等于 *x* 则结果为 `True`，否则为 `False`    | for x in s: print(x) # 迭代                                  |
| `x not in s`           | 如果 *s* 中的某项等于 *x* 则结果为 `False`，否则为 `True`    |                                                              |
| `s + t`                | *s* 与 *t* 相拼接                                            |                                                              |
| `s * n` 或 `n * s`     | 相当于 *s* 与自身进行 *n* 次拼接                             | 注意序列 *s* 中的项并不会被拷贝；它们会被多次引用，修改任一项，其余项也跟着变。 |
| `s[i]`                 | *s* 的第 *i* 项，起始为 0                                    | i 为负数时，从后往前取，最后一位为 -1。                      |
| `s[i:j]`               | *s* 从 *i* 到 *j* 的切片                                     | 左闭右开区间：[i, j)                                         |
| `s[i:j:k]`             | *s* 从 *i* 到 *j* 步长为 *k* 的切片                          | 步长 k 为负时，方向取值。                                    |
| `len(s)`               | *s* 的长度                                                   |                                                              |
| `min(s)`               | *s* 的最小项                                                 |                                                              |
| `max(s)`               | *s* 的最大项                                                 |                                                              |
| `s.index(x[, i[, j]])` | *x* 在 *s* 中首次出现项的索引号（索引号在 *i* 或其后且在 *j* 之前） |                                                              |
| `s.count(x)`           | *x* 在 *s* 中出现的总次数                                    |                                                              |

#### 3.2、集合（set）

> 集合（Set）是一种无序、可变的数据类型，用于存储唯一的元素。
>
> 可以进行交集、并集、差集等常见的集合操作。
>
> 集合使用大括号 {} 表示，元素之间用逗号 **,** 分隔。

```python
# 定义空集合，
# 不能用 {}，因为 {} 是创建空字典的
>>> set1, set2 = set(), {}
>>> type(set1), type(set2)
(<class 'set'>, <class 'dict'>)

>>> s = {'C', 'C++', 'Java', 'Python', 'Ruby', 'Golang'}

# 添加一个元素，忽略已存在
>>> s.add('PHP') 

# 添加一批元素，可以是 list、tuple、set、dict（只添加 keys），忽略已存在
>>> s.update(('HTML', 'Javascript', 'PHP')) 
>>> s
{'C++', 'Java', 'Ruby', 'Golang', 'PHP', 'Javascript', 'C', 'Python', 'HTML'}

# 移除一个元素，不存在时报错
>>> s.remove('HTML')
>>> s.remove('HTML')
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
KeyError: 'HTML'

# 移除元素，不存在时不报错
>>> s.discard('HTML')

# 弹出第一个元素
>>> s
{'C++', 'Java', 'Ruby', 'Golang', 'PHP', 'Javascript', 'C', 'Python'}
>>> s.pop()
'C++'
>>> s.pop()
'Java'

# 集合元素个数
>>> len(s)
6

# 清空
>>> s.clear()

# 遍历
>>> for x in s: 
...     print(x, end=' ')
... else:
...     print()

# ===================================== 

# 差集
>>> print({1, 2, 3} - {1, 2, 4})
{3}

# 并集
>>> print({1, 2, 3} | {1, 2, 4})
{1, 2, 3, 4}

# 交集
>>> print({1, 2, 3} & {1, 2, 4})
{1, 2}

# 集合中不同时存在的元素
>>> print({1, 2, 3} ^ {1, 2, 4})
{3, 4}
```

> 集合方法

| 方法                          | 描述                                                         |
| :---------------------------- | :----------------------------------------------------------- |
| add()                         | 为集合添加元素                                               |
| clear()                       | 移除集合中的所有元素                                         |
| copy()                        | 拷贝一个集合                                                 |
| difference()                  | 返回多个集合的差集                                           |
| difference_update()           | 移除集合中的元素，该元素在指定的集合也存在。                 |
| discard()                     | 删除集合中指定的元素                                         |
| intersection()                | 返回集合的交集                                               |
| intersection_update()         | 返回集合的交集。                                             |
| isdisjoint()                  | 判断两个集合是否包含相同的元素，如果没有返回 True，否则返回 False。 |
| issubset()                    | 判断指定集合是否为该方法参数集合的子集。                     |
| issuperset()                  | 判断该方法的参数集合是否为指定集合的子集                     |
| pop()                         | 随机移除元素                                                 |
| remove()                      | 移除指定元素                                                 |
| symmetric_difference()        | 返回两个集合中不重复的元素集合。                             |
| symmetric_difference_update() | 移除当前集合中在另外一个指定集合相同的元素，并将另外一个指定集合中不同的元素插入到当前集合中。 |
| union()                       | 返回两个集合的并集                                           |
| update()                      | 给集合添加元素                                               |
| len()                         | 计算集合元素个数                                             |

> 集合推导式

```python
# 语法
{ expression for item in Sequence }
或
{ expression for item in Sequence if conditional }


# 计算数字 1,2,3 的平方数
>>> setnew = {i**2 for i in (1,2,3)}
>>> setnew
{1, 4, 9}

# 判断不是 abc 的字母并输出
>>> a = {x for x in 'abracadabra' if x not in 'abc'}
>>> a
{'d', 'r'}
```



#### 3.3、字典（dict）

> 列表是有序的对象集合，字典是无序的对象集合。
>
> 两者之间的区别在于：字典当中的元素是通过键来存取的，而不是通过偏移存取。
>
> 字典是一种映射类型，字典用 {} 标识，它是一个无序的 **键(key) : 值(value)** 的集合。
>
> 字典中，键（key）必须使用不可变类型，且键（key）必须是唯一的。
>
> 键必须不可变，所以可以用数字，字符串或元组充当，而用列表就不行。

```python
# 定义空字典
>>> dict1, dict2 = dict(), {}

# 定义字典：
>>> user = {'name': 'ycz', 'age': 28}

# 增加/修改字典键值对
user['age'] = 18
user['score'] = 0

# 获取字段值
>>> user['age']
18

# key 不存在时报错
>>> user['asdfg']
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
KeyError: 'asdfg'

# 删除字典键值
>>> del(user['score'])

# 字典内置函数
>>> type(user) 
<class 'dict'>
>>> len(user)
2
>>> str(user) 
"{'name': 'ycz', 'age': 18}"


# 字典方法
>>> user.keys()
dict_keys(['name', 'age'])
>>> user.values()
dict_values(['ycz', 18])
>>> user.items()  
dict_items([('name', 'ycz'), ('age', 18)])
```

> 字典方法

| 函数                               | 描述                                                         |
| :--------------------------------- | :----------------------------------------------------------- |
| dict.clear()                       | 删除字典内所有元素                                           |
| dict.copy()                        | 返回一个字典的浅复制                                         |
| dict.fromkeys()                    | 创建一个新字典，以序列seq中元素做字典的键，val为字典所有键对应的初始值 |
| dict.get(key, default=None)        | 返回指定键的值，如果键不在字典中返回 default 设置的默认值    |
| key in dict                        | 如果键在字典dict里返回true，否则返回false                    |
| dict.items()                       | 以列表返回一个视图对象                                       |
| dict.keys()                        | 返回一个视图对象                                             |
| dict.setdefault(key, default=None) | 和get()类似, 但如果键不存在于字典中，将会添加键并将值设为default |
| dict.update(dict2)                 | 把字典dict2的键/值对更新到dict里                             |
| dict.values()                      | 返回一个视图对象                                             |
| dict.pop(key[,default\])           | 删除字典 key（键）所对应的值，返回被删除的值。               |
| dict.popitem()                     | 返回并删除字典中的最后一对键和值。                           |

> 字典推导式

```python
# 语法：
{ key_expr: value_expr for value in collection }
或
{ key_expr: value_expr for value in collection if condition }


# 将列表中各字符串值为键，各字符串的长度为值，组成键值对
>>> listdemo = ['Google','Runoob', 'Taobao']
>>> newdict = {key:len(key) for key in listdemo}
>>> newdict
{'Google': 6, 'Runoob': 6, 'Taobao': 6}


# 以数字为键，数字的平方为值来创建字典
>>> {x: x**2 for x in (2, 4, 6)}
{2: 4, 4: 16, 6: 36}
```



#### 3.4、字节序（bytes）

> bytes 类型表示的是不可变的二进制序列（byte sequence）。
>
> 与字符串类型不同的是，bytes 类型中的元素是整数值（0 到 255 之间的整数），而不是 Unicode 字符。
>
> bytes 类型通常用于处理二进制数据，比如图像文件、音频文件、视频文件等等。在网络编程中，也经常使用 bytes 类型来传输二进制数据。

```python
# 使用 b 前缀创建 bytes 对象
>>> bytes1 = b"hello"
>>> type(bytes1)
<class 'bytes'>

# 拼接
>>> bytes1 + b' world'
b'helloworld'

# bytes 类型中的元素是整数值，因此在进行比较操作时需要使用相应的整数值。
# ord() 函数用于将字符转换为相应的整数值。
>>> bytes1[0] == 'h'
False
>>> bytes1[0] == ord('h')
True

```

#### 3.5、数据类型转换

| 函数                   | 描述                                                |
| :--------------------- | :-------------------------------------------------- |
| int(x [,base])         | 将 x 转换为一个整数                                 |
| float(x)               | 将 x 转换到一个浮点数                               |
| complex(real [,imag\]) | 创建一个复数                                        |
| str(x)                 | 将对象 x 转换为字符串                               |
| repr(x)                | 将对象 x 转换为表达式字符串                         |
| eval(str)              | 用来计算在字符串中的有效Python表达式,并返回一个对象 |
| tuple(s)               | 将序列 s 转换为一个元组                             |
| list(s)                | 将序列 s 转换为一个列表                             |
| set(s)                 | 转换为可变集合                                      |
| dict(d)                | 创建一个字典。d 必须是一个 (key, value)元组序列。   |
| frozenset(s)           | 转换为不可变集合                                    |
| chr(x)                 | 将一个整数转换为一个字符                            |
| ord(x)                 | 将一个字符转换为它的整数值                          |
| hex(x)                 | 将一个整数转换为一个十六进制字符串                  |
| oct(x)                 | 将一个整数转换为一个八进制字符串                    |

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
# 语法：
while <expr>:
    <statement(s)>
else:
    # 循环结束后执行的代码
    <additional_statement(s)>

num = 10

while num > 0:
    print(num)
    num -= 1
```

#### （3）for

```python
# 语法
for <variable> in <sequence>:
    # 循环主体
    <statements>
else:
    # 循环结束后执行的代码
    <statements>
    

letter = "abcdefghijklmnopqrstuvwxyz"
for byte in letter:
    print(byte)

list = ['hello', 'world', 'how', 'are', "you"]    
for word in list:
    print(word)
```

#### （4）range

> 内置函数 [`range()`](https://docs.python.org/zh-cn/3/library/stdtypes.html#range) 用于生成等差数列

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

#### （6）match

```python
def http_error(status):
    match status:
        case 400:
            return "Bad request"
        case 401 | 403 | 404:
            return "Not allowed"
        case 404:
            return "Not found"
        case 418:
            return "I'm a teapot"
        # _ 被作为通配符，并必定会匹配成功
        case _:
            return "Something's wrong with the internet"
```

#### （7）pass

pass是空语句，是为了保持程序结构的完整性。

pass 不做任何事情，一般用做占位语句，如下实例：

```python
>>> while True:
...     pass  # 等待键盘中断 (Ctrl+C)

# 最小的类
>>> class MyEmptyClass:
...     pass

# 
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

#### （5）匿名函数（lambda）

```python
# 可以包含零个或多个参数
lambda [arguments, ...]: 函数体（只能写一行代码）

func_add = lambda x, y: x + y
print(func_add(1, 2))


# lambda 函数通常与内置函数如 map()、filter() 和 reduce() 一起使用

numbers = [1, 2, 3, 4, 5]
squared = list(map(lambda x: x**2, numbers))
print(squared)  # 输出: [1, 4, 9, 16, 25]

# 与 filter() 一起，筛选偶数
numbers = [1, 2, 3, 4, 5, 6, 7, 8]
even_numbers = list(filter(lambda x: x % 2 == 0, numbers))
print(even_numbers)  # 输出：[2, 4, 6, 8]


# 使用 reduce() 计算一个序列的累积乘积
from functools import reduce
 
numbers = [1, 2, 3, 4, 5]
 
# 使用 reduce() 和 lambda 函数计算乘积
product = reduce(lambda x, y: x * y, numbers)
 
print(product)  # 输出：120
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


if __name__ == '__main__':
    CRS = crs()
    CRS(100, True)
    CRS(200, True)
    CRS(300, True)
    CRS(100)
```

#### （7）装饰器

> 装饰器：在不破坏被装饰对象源代码和调用方式的前提下，为被装饰对象添加额外的功能。
>
> 被装饰对象可以是：函数、方法、类。
>
> 装饰器应用场景：日志记录、性能分析、权限控制、缓存、事务处理。
>
> 装饰器是一种函数，它接受一个函数作为参数，并返回一个新的函数或修改原来的函数。

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

- 装饰器的参数

```python
# 重复执行的装饰器
def repeat(num_times):
    def decorator(func):
        def wrapper(*args, **kwargs):
            for _ in range(num_times):
                func(*args, **kwargs)
        return wrapper
    return decorator


# 执行 3 次
@repeat(3)
def say_hello():
    print("Hello!")

say_hello()
```

> 类装饰器

类装饰器（Class Decorator）是一种用于动态修改类行为的装饰器，它接收一个类作为参数，并返回一个新的类或修改后的类。

类装饰器可以用于：

- 添加/修改类的方法或属性
- 拦截实例化过程
- 实现单例模式、日志记录、权限检查等功能

**类装饰器有两种常见形式：**

- 函数形式的类装饰器（接收类作为参数，返回新类）
- 类形式的类装饰器（实现 **__call__** 方法，使其可调用）



### 6、异常处理

> 捕获异常

```python
try:
    print("try execute...")
    5/0
except Exception as e:
    print("Capture exceptions:", e)
except Exception1:
    print("...")
else:
    print("[else] optional: No exceptions")
finally:
    print("[finally] optional: 有无异常都会执行")

print("execute end...")

$ python ...
try execute...
Capture exceptions: division by zero
[finally] optional: 有无异常都会执行
execute end...
```

> 抛出异常

```python
# 语法
raise [Exception [, args [, traceback]]]

raise Exception('出错了...')
```



### 7、包与模块

#### （1）模块

```python
# vim hello.py

def hello(name):
    print(f"hello {name}")


# 一个模块被另一个程序第一次引入时，其主程序将运行。
# 如果我们想在模块被引入时，模块中的某一程序块不执行，
# 我们可以用 __name__ 属性来使该程序块仅在该模块自身运行时执行。
# __name__ 是一个内置变量，表示当前模块的名称。

# __main__ 仅在当前模块运行时执行（用作单元测试）
if __name__ == '__main__':
    hello("ycz")
else:
    # 如果模块是被直接运行，__name__ 的值为 __main__。
    # 如果模块是被导入的，__name__ 的值为当前模块名。
    print('被其它模块导入')
```

#### （2）包

```
pkg/              # pkg 包，包由一个个模块组成，模块名同文件名
  ├─ __init__.py  # 【必须】存在该文件时，才叫 python 包，否则是普通文件夹
  ├─ mathx.py     # 数学模块
  ├─ cryptox.py   # 加密模块
  └─ ...
call_pkg.py   # 调用包
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

> `mathx.py`

```python
# 当使用 from mathx import * 导入时，
# 只能导入 __all__ 变量列表中指定的方法（__all__ 为空时，无限制）
# 其它导入方式不受限制，如 import hello | from hell import internal
__all__ = ["add", "sub", "mul", "dlv"]


def add(x, y: int):
    """
    :return: x + y
    """
    return x + y


def sub(x, y: int):
    """
    :return: x - y
    """
    return x - y


def mul(x, y: int):
    """
    :return: x * y
    """
    return x * y


def dlv(x, y: int):
    """
    :return: x / y
    """
    return x / y


def internal():
    print("internal function...")


# 本包单元测试用（其他包引用时，不会执行）
if __name__ == '__main__':
    print("Unit testing add():", add(1, 1))
    internal()

"""
# 执行单元测试：
$ python mathx.py
Unit testing add(): 2
internal function...
"""
```

> `call_pkg.py`

```python
from pkg.mathx import *

print("x + y =", add(1, 1))
print("x - y =", sub(5, 3))
print("x * y =", mul(2, 2))
print("x / y =", dlv(5, 2))
internal()


"""
# 执行调包案例：
$ python call_pkg.py
x + y = 2
x - y = 2
x * y = 4
x / y = 2.5
Traceback (most recent call last):
  File "...\call_pkg.py", line 7, in <module>
    internal()
    ^^^^^^^^
NameError: name 'internal' is not defined
"""
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
def 函数方法名(var: Type [| Type | ...], var: Type [| Type | ...], ...) -> Type [| Type | ...]:
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

    # 类方法第一个参数必须为 self
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
    # 【私有成员变量】 __ 开头的成员变量
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

    # 【私有成员方法】 __ 开头的成员方法
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

> 类的专有方法：

```
__init__ : 构造函数，在生成对象时调用
__del__ : 析构函数，释放对象时使用
__repr__ : 打印，转换
__setitem__ : 按照索引赋值
__getitem__: 按照索引获取值
__len__: 获得长度
__cmp__: 比较运算
__call__: 函数调用
__add__: 加运算
__sub__: 减运算
__mul__: 乘运算
__truediv__: 除运算
__mod__: 求余运算
__pow__: 乘方
```



### 1、继承

```python
# 子类（派生类 DerivedClassName）会继承父类（基类 BaseClassName）的属性和方法。
# BaseClassName（实例中的基类名）必须与派生类定义在一个作用域内。
# 基类定义在另一个模块中时：
# class DerivedClassName(modname.BaseClassName):

# 定义父类
class Father:
    def my_name(self):
        print("My name is father")


# 定义母类
class Mother:
    def my_name(self):
        print("My name is mother")


# 定义子类 1
# 多继承优先级：Parent1 > Parent2 > ...
class Child1(Father, Mother):
    pass


# 定义子类 2
# 可以在子类重写父类的成员和方法
class Child2(Father, Mother):
    def my_name(self):
        print("My name is child-2")

    def call_mother(self):
        # 通过父类名调用父类成员和方法
        Mother.my_name(self)


if __name__ == '__main__':
    # 使用多继承时，按继承父类方法时的顺序表（MRO）调用
    child1 = Child1()
    child1.my_name()  # My name is father

    # 重写父类方法
    child2 = Child2()
    child2.my_name()  # My name is child-2

    # super() 调用父类（超类）的方法（多继承按 MRO 调用）
    super(Child2, child2).my_name()  # My name is father
    # 这里想要调用 MRO 中第二个方法，只能在子类中通过父类名调用父类的方法
    child2.call_mother()  # My name is mother
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



### 2、虚拟环境（venv）

```
usage: 
  venv [-h] [--system-site-packages] [--symlinks | --copies] [--clear] [--upgrade] [--without-pip] [--prompt PROMPT] [--upgrade-deps] ENV_DIR [ENV_DIR ...]

Creates virtual Python environments in one or more target directories.

positional arguments:
  ENV_DIR               A directory to create the environment in.

options:
  -h, --help            show this help message and exit
  --system-site-packages
                        Give the virtual environment access to the system site-packages dir.
  --symlinks            Try to use symlinks rather than copies, when symlinks are not the default for the platform.
  --copies              Try to use copies rather than symlinks, even when symlinks are the default for the platform.
  --clear               Delete the contents of the environment directory if it already exists, before environment creation.
  --upgrade             Upgrade the environment directory to use this version of Python, assuming Python has been upgraded in-place.
  --without-pip         Skips installing or upgrading pip in the virtual environment (pip is bootstrapped by default)
  --prompt PROMPT       Provides an alternative prompt prefix for this environment.
  --upgrade-deps        Upgrade core dependencies: pip setuptools to the latest version in PyPI
```

```sh
# 创建虚拟环境的目录，一般为 .venv
$ python -m venv .venv

# 激活虚拟环境
$ .venv/Scripts/activate

# (.venv) 虚拟环境标识
(.venv) ... >

# 列出虚拟环境中已安装的包
(.venv) ... > pip list
Package    Version
---------- -------
pip        23.1.2 
setuptools 65.5.0

# 在虚拟环境下执行 python
(.venv) ... > python main.py

# 退出虚拟环境
(.venv) ... > deactivate
```

> 依赖库共享

```sh
# 以 requirements 格式输出已安装的软件包
$ pip freeze > requirements

# 查看依赖
$ cat requirements

appdirs==1.4.4
attrs==25.3.0
bcrypt==4.2.0
blinker==1.9.0
certifi==2025.7.14
cffi==1.17.1
charset-normalizer==3.3.2
click==8.2.1
colorama==0.4.6
dnspython==2.6.1
Flask==3.1.1
future==1.0.0
greenlet==3.2.3
h11==0.16.0
idna==3.7
importlib_metadata==8.7.0
itsdangerous==2.2.0
Jinja2==3.1.6
jsonpath==0.82.2
lxml==6.0.0
MarkupSafe==3.0.2
numpy==2.3.1
outcome==1.3.0.post0
pandas==2.3.1
psutil==7.0.0
pycparser==2.22
pyee==11.1.1
PyHive==0.7.0
pymongo==4.7.1
PyMySQL==1.1.1
pynum==0.0.1
pyppeteer==2.0.0
PySocks==1.7.1
python-dateutil==2.9.0.post0
pytz==2025.2
requests==2.31.0
selenium==4.34.2
six==1.17.0
sniffio==1.3.1
sortedcontainers==2.4.0
SQLAlchemy==2.0.41
tabula==1.0.5
tqdm==4.67.1
trio==0.30.0
trio-websocket==0.12.2
typing_extensions==4.14.1
tzdata==2025.2
urllib3==1.26.20
websocket-client==1.8.0
websockets==10.4
Werkzeug==3.1.3
wsproto==1.2.0
zipp==3.23.0

# 从给定的 requirements 文件安装
$ pip install -r requirements
Collecting appdirs==1.4.4 (from -r requirements (line 1))
  Downloading appdirs-1.4.4-py2.py3-none-any.whl (9.6 kB)
...
```

## 五、Web 框架

 主流框架核心对比

| **框架**    | **定位**       | **核心优势**                                                 | **劣势**                                                     | **典型应用场景**                         |
| :---------- | :------------- | :----------------------------------------------------------- | :----------------------------------------------------------- | :--------------------------------------- |
| **Django**  | 全栈式开发框架 | 内置ORM、Admin、认证等20+组件，开箱即用<br />完善的文档和社区，企业级扩展能力（如DRF）<br />强安全性（CSRF/SQL注入防护） | 重量级，小型项目冗余<br />性能较低（QPS≈2.5k）<br />异步支持弱于新框架 | CMS/ERP、电商后台、数据驱动型应用        |
| **Flask**   | 微服务基石     | 极简内核（200KB），灵活扩展<br />低学习曲线，适合快速原型<br />自由组合库（如SQLAlchemy+WTForms） | 无内置功能（需手动实现安全/Admin）<br />性能中等（QPS≈3.8k）<br />大型项目结构易混乱 | REST API微服务、小型应用、数据可视化平台 |
| **FastAPI** | 高性能API框架  | 极致性能（QPS≈76k），异步支持<br />自动OpenAPI文档+类型安全验证（Pydantic）<br />现代开发体验（Python 3.7+类型提示） | 生态成熟度较低（扩展库较少）<br />无内置Admin/ORM，需额外集成 | 高并发API、实时服务、机器学习部署        |
