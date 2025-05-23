---
title: python 基础语法
excerpt: "python 基础语法"
date: 2025-05-21 9:20:57
tag: [python]
categories: [python]
---

#### 运算符
```python
# '/'  除法（浮点数结果）
print(10/3) //3.3333333333333335

# '//'  除法（整数结果）
print(10//3) //3

# '%'  取余数
print(10%3) //1

# '**'  幂运算
print(10**3) //1000

```
> 赋值运算符
1. =：简单赋值
2. +=：加后赋值
3. -=：减后赋值
4. *=：乘后赋值
5. /=：除后赋值
6. //=：整除后赋值
7. %=：取余后赋值
8. **=：幂运算后赋值

> 逻辑运算符
1. and：逻辑与
2. or：逻辑或
3. not：逻辑非

> 成员运算符
1. in：如果元素在序列中则返回 True
2. not in：如果元素不在序列中则返回 True

> 身份运算符
1. is：如果两个对象指向同一个内存地址则返回 True
2. is not：如果两个对象不指向同一个内存地址则返回 True

:=：海象运算符（Python 3.8 引入），可以在表达式内部为变量赋值

#### 流程控制
```python
if condition:
    # 如果 condition 为 True，则执行这里的代码
    
for variable in iterable:
    # 对每个元素执行操作

fruits = ["apple", "banana", "cherry"]
for fruit in fruits:
    print(fruit)

#还可以结合 range() 使用，生成数字序列：
for i in range(5):
    print(i)  # 输出 0 到 4
    
while condition:
    # 只要 condition 为 True，就一直执行这段代码

#break：退出当前循环。
#continue：跳过当前循环的剩余部分，进入下一次循环。
#pass：占位符，不做任何事情。

for num in range(2, 10):
    if num % 2 == 0:
        print(num, '是偶数')
        continue
    print(num, '是奇数')

```

#### 异常处理

```python
try:
    # 可能引发异常的代码块
    ...
except ExceptionType:
    # 处理特定类型的异常
    ...
    
#如果没有异常发生，可以使用 else 块来执行一些代码。
try:
    result = 10 / 2
except ZeroDivisionError:
    print("不能除以零")
else:
    print("结果是:", result)

#使用 finally 子句
try:
    file = open("example.txt", "r")
    content = file.read()
except FileNotFoundError:
    print("文件未找到")
finally:
    print("执行清理操作")

#自定义异常，可以通过继承 Exception 类来自定义异常类型。
class MyCustomError(Exception):
    pass

try:
    raise MyCustomError("这是一个自定义错误")
except MyCustomError as e:
    print(e)
```

#### 数据结构
> 列表

1. 添加元素：append()、extend()、insert()
2. 删除元素：remove()、pop()、del
3. 排序：sort()
4. 反转：reverse()
5. 切片：my_list[start:end]

```python
my_list = [1, "two", 3.0]

mylist = [1,9,3,5,7,2,8,4,6]
mylist.sort()
```


> 推导式

[expression for item in iterable if condition]

```python
# 生成平方数列
squares = [x**2 for x in range(5)]
print(squares)  # 输出: [0, 1, 4, 9, 16]

# 带条件筛选的列表推导式
even_numbers = [x for x in range(10) if x % 2 == 0]
print(even_numbers)  # 输出: [0, 2, 4, 6, 8]
```

> 字典

1. 访问元素：my_dict[key]
2. 添加/修改元素：my_dict[key] = value
3. 删除元素：del my_dict[key]、pop()
4. 遍历：for key in my_dict:、for key, value in my_dict.items():

```python
dict = {"name":"licon","age":18}
print(dict["name"])

del dict["name"] #删除字典的键位
print(dict)

for key in dict:
    print(key)

for key,value in dict.items():
    print(key,value)
```

> 推导式

{key_expr: value_expr for item in iterable if condition}

```python
# 创建字典：数字 -> 平方
squares_dict = {x: x**2 for x in range(5)}
print(squares_dict)  # 输出: {0: 0, 1: 1, 2: 4, 3: 9, 4: 16}

# 筛选偶数键值对
even_dict = {x: x**2 for x in range(10) if x % 2 == 0}
print(even_dict)  # 输出: {0: 0, 2: 4, 4: 16, 6: 36, 8: 64}
```


> 元组

1. 不可变，因此不能添加、删除或修改元素。
2. 可以用作字典的键（如果其中的元素也是不可变的）。

```python
my_tuple = (1, "two", 3.0)
print(my_tuple.count(1)) #  返回元素出现的次数
print(my_tuple.index("two")) # 返回元素在元组中的索引
```


> 集合

1. 添加元素：add()
2. 删除元素：remove()、discard()
3. 集合运算：union()、intersection()、difference()
4. 检查成员：in

```python
set={1,2,2,3,3,5,6,7}
set.union() # 返回并集
print(set)

set2={2,3,4}
print(set.intersection())  # 返回交集
```

> 推导式

{expression for item in iterable if condition}

```python
# 生成不重复的平方集合
unique_squares = {x**2 for x in [1, -1, 2, -2, 3]}
print(unique_squares)  # 输出: {1, 4, 9}

# 筛选偶数的集合
even_set = {x for x in range(10) if x % 2 == 0}
print(even_set)  # 输出: {0, 2, 4, 6, 8}

```


> 生成器推导式

(expression for item in iterable if condition)

```python
# 生成器表达式
gen = (x**2 for x in range(5))
print(gen)  # 输出: <generator object <genexpr> at 0x...>

# 可以逐个取值
for val in gen:
    print(val)

# 或者转为列表
print(list((x**2 for x in range(5))))  # 输出: [0, 1, 4, 9, 16]

```
>  多层嵌套推导式

[expression for item1 in iterable1 for item2 in iterable2 if condition]

```python
# 嵌套循环
pairs = [(x, y) for x in [1, 2] for y in ['a', 'b']]
print(pairs)  # 输出: [(1, 'a'), (1, 'b'), (2, 'a'), (2, 'b')]

# 带条件的嵌套推导式
filtered_pairs = [(x, y) for x in range(3) for y in range(3) if x != y]
print(filtered_pairs)  # 输出: [(0, 1), (0, 2), (1, 0), (1, 2), (2, 0), (2, 1)]

```

> 迭代器（Iterator）

迭代器是一个对象，实现了 __iter__() 和 __next__() 方法。
当调用 next() 方法时，迭代器会返回下一个元素。
当迭代器没有更多元素时，next() 方法会抛出 StopIteration 异常。

```python
#每次调用 next() 获取下一个元素。
#所有元素存储在内存中。

my_list = [1, 2, 3]
it = iter(my_list)

print(next(it))  # 输出 1
print(next(it))  # 输出 2
print(next(it))  # 输出 3
# print(next(it))  # 抛出 StopIteration

```

#### 类和对象

```python

#在 Python 中，self 是类中方法的第一个参数，它的作用是 指向调用该方法的实例本身。
#这是 Python 的一种设计机制，用于实现面向对象编程（OOP）中的 封装性 和 多态性

#如果没有 self，Python 无法知道这个函数是否属于某个类的实例。
#self 帮助 Python 把函数绑定到类的实例上，变成“实例方法”。


#在 Python 中，魔术方法（Magic Methods） 也称为 双下划线方法（Dunder Methods），
#是 Python 提供的一类特殊方法，用于实现对象的内置行为，例如初始化、字符串表示、运算符重载、迭代等。
#它们以双下划线 __ 开头和结尾，例如 __init__()、__str__()、__add__() 等。

class User:
    def __init__(self, name, age):  #构造函数
        self.name = name
        self.age = age

    def  __str__(self):    # 返回对象的字符串表示
        return f"User {self.name} is {self.age} years old."   #f 表示这是一个 f-string，是 Python 3.6+ 引入的新特性；
                                                              #{self.name} 是一个占位符，运行时会被对象的 name 属性值替换。
    
    def  __eq__(self, other):  # 比较两个对象是否相等
        return self.name == other.name and self.age == other.age
        
        
#如果想把某些属性给隐藏起来，不想让外界直接访问，可以在属性名前面加两个下划线
#可以添加set和get方法
class User:
    def __init__(self, name, age):  #构造函数
        self.name = name
        self.age = age
        self.__secret = "This is a secret."
    
    def set(self,secret):
        self.__secret = secret

    def get(self):
        return self.__secret

user = User("licon",18)
        
#创建用于计算的属性
#python中使用@property将一个方法转换为属性，从而实现用于计算的属性，
#将方法转换为属性后，可以直接通过方法名来访问，而不需要加括号。

class User:
    def __init__(self,name,age):
        self.name = name
        self.age = age

    @property
    def func(self):
        if self.age < 150 and self.age > 0:
            print(self.age)
        else:
            print("想啥呢？")

user = User("licon",18)
user.func


```

