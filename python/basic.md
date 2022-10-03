# 基础

## 数据类型

### 字符串

- .title() #首字母大写
- .upper() .lower()
- `+` #拼接
- `\t \n` #制表符、换行符
- strip() rstrip() lstrip()
- `*` #重复多少次

### 数字

- `**` #乘方
- str() #转换成字符串

### 布尔

- 字符串非空 为 True
- 集合非空 为 True

## 列表

- [] 表示
- .append() .insert(index, e)
- `del list[0]`
- pop() #弹出最后一个元素，并返回，也可指定 index
- .remove() #根据值删除
- len(list) #列表长度
- .reverse() #反转列表元素
- .sort() #列表排序 .sort(reverse=True) #反向排序
- new_list = sorted(list) #排序，不改变原列表排序
- 索引 -1，返回只有一个元素
- `if x in list  if x not in list` 判断元素在列表中
- `if list:` 判断列表不是空的

```py
for it in it_list:
    xxx
```

- range(start, end) 函数创建列表
- range(start, end, step)

```py
list(range(1, 6))
```

- 数字列表函数 min max sum

- 列表解析

```py
list = [value ** 2 for value in range(1, 11)]
```

- 切片
  - `lits[:]` 全部列表 `[:4]` `[4:]` 从头开始、直到末尾
  - 值为索引
  - 返回一个列表，可进行遍历等操作
  - `list2 = list[:]` 复制列表

## 元组

- 不可变列表 `()` 表示

## 字典

- 一系列键值对 `{}` 表示，`:` 分隔
- 访问 `dict[键]`
- `del dict[xx]` 删除键值
- .items() 返回键值列表 .keys() 返回键列表 .values() 返回值列表
- 遍历字典名默认返回 key 列表

- 遍历键值

```py
for key, value in users.items():

按顺序遍历

for name in sorted(favorite.keys()):
```

## set 集合

元素不重复

```py
for language in set(favorite_languges.values()):
```

## 用户输入

`input()` 暂停程序，等待用户输入，接受的值为字符串

## while 循环 处理集合

```py
while unconfirmed_users:
    current_user = uncnofirmed_users.pop()
    confirmed_users.append(current_user)
```

## 函数

定义函数

```py
def greet():
    print('hello')

greet()
```

### 参数

- 普通参数（位置参数），根据位置确定参数的值

- 关键字参数
  - 传递参数名和参数值，无需考虑顺序
  - 可在函数参数定义中直接指定默认值，用 = 链接，放在位置参数的后面
  - 使用默认值，可以在调用的时候不指关键字参数
  - 关键字参数也可当作位置参数进行调用

```py
def greet(name, age=18):
    print('hello'+name+':'+age)

greet(name='a', age='12')
greet(name='b')
```

### 任意数量实参

- `*varname` 作为参数是，函数会将转化成一个元组
- 要放在最后，python 会先匹配位置实参和关键字实参，在将余下的实参收集到最后一个形参中

```python
def make_pizza(*toppings):
    xxxxx
```

- 可将任意数量的实参写成键值对，函数会转化为一个字典。

```python
def build_profile(first, last, **user_info):
```

### 导入模块

- 导入整个模块 `import x` 调用方式为 `x.abc()`
- `from module_name import *` 可以直接使用函数名称来调用函数，这种方法可能会与现有项目中的名称相同，进而将函数覆盖
- `from module_name import func_1, func_2
- `from module_name import func_name as fn`
- `import x as md`

## 类

- 类中的函数称为方法
- `__init__` 创建新实例时会调用，self 形参必不可少，自动传递

```py
class Dog():
    def __init__(self, x, y):
```

创建实例

```py
mydog = Dog('x', 'y')
```

### 继承

父类必须写在子类同一文件的前面，括号内指定父类名称，`__init__()` 方法中接受创建父类实例的信息。

## 文件

```py
with open(filename) as file_object:
    for line in file_object:
        print(line.rstrip())
    lines = file_object.readlines()
```

## 异常

```py
try:
    xxx
except xxxError:
    xxx
else:
```

## 测试

类必须继承 unittest.TestCase 类