---
title: python 基础语法
excerpt: "python 基础语法"
date: 2025-05-21 9:20:57
tag: [python]
categories: [python]
---

#### 运算符
```python
// '/'  除法（浮点数结果）
print(10/3) //3.3333333333333335

// '//'  除法（整数结果）
print(10//3) //3

// '%'  取余数
print(10%3) //1

// '**'  幂运算
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

