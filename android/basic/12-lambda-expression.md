# Lambda 表达式

## Jack

Jack 局限性

- 不能使用 data binding 库
- 不能使用 Instant Run 功能
- 基于 .class  文件的代码分析等工具无法使用
- 不支持 multidex backport 和 JSoup Library

Jack 会弃用

```
android {
    compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_8
        targetCompatibility JavaVersion.VERSION_1_8
    }
}
```

```java
if (sortAscending) {
    Collections.sort(temp, Video::compareTo);
} else {
    Collections.sort(temp, (one, two) -> two.compareTo(one));
}
```

## 函数式接口 Functional Interface

只拥有一个方法的接口称为函数式接口 Single Abstract Method

JavaSE 7 中存在的函数式接口

- java.lang.Runnable
- java.util.concurrent.Callable
- java.sercurity.PrivilegedAction
- java.util.Comparator
- java.io.FileFilter
- java.beans.PropertyChangeListener

JavaSE 8 中新增加了一个包 `java.util.function`，包含了常用的函数式接口

- `Predicate<T>` 接收`T`对象，返回 boolean
- `Consumer<T>` 接收`T`对象，不返回值
- `Function<T， R>` 接收`T`对象，返回`R`对象
- `Supplier<T>` 提供`T`对象（例如工厂），不接收值
- `UnaryOperator<T>` 接收`T`对象，返回`T`对象
- `BinaryOperator<T>` 接收两个`T`对象，返回`T`对象

还有一些针对原始类型的特化函数式接口，一些针对多个参数的函数式接口

## lambda expressions

由参数列表、`->`、函数体组成

- 函数体，可以是一个表达式，也可以是一个代码块。
  - 表达式会被执行并返回执行结果
  - 代码块中的语句会被依次执行

## 目标类型 Target typing

不同上下文，lambda 表达式的类型也不同，编译器负责推导其类型

- `T`是一个函数式接口
- 表达式参数和`T`的方法参数在数量和类型上一一对应
- 表达式的返回值和`T`的返回值相兼容
- 表达式内所抛出的异常和`T`的方法`throws`类型相兼容

lambda 的参数类型可以从目标类型（函数式接口）中得出，因此不用声明类型，当参数为一个时，括号也可以省略

## 目标类型的上下文 Contexts for target typing

带有目标类型的上下文

- 变量声明
- 赋值
- 返回语句
- 数组初始化器
- 方法和构造方法的参数
- lambda 表达式函数体
- 条件表达式`? :`
- 转型表达式

```java
Comparator<String> c;
c = (String s1, String s2) -> s1.compareToIgnoreCase(s2);

public Runnable toDoLater() {
  return () -> {
    System.out.println("later");
  }
}

filterFiles(new FileFilter[] {
              f -> f.exists(), f -> f.canRead(), f -> f.getName().startsWith("q")
            });
```

对于参数类型的推导复杂一些，涉及到重载解析和参数类型推导

如果 lambda 表达式具有显示类型（参数类型被显示指定），编译器就可以直接使用表达式的返回类型

如果 lambda 表达式具有隐式类型（参数类型被推导而知），重载解析就会忽略 lambda 表达式函数体而只依赖 lambda 表达式的参数数量

如果在解析方法声明时存在二义性，就需要利用转型或显示 lambda 表达式来提供更多的类型信息

如果 lambda 表达式的返回类型依赖于其参数的类型，那么 lambda 表达式函数体有可能可以给编译器提供额外的信息，以便其推导参数类型

## 词法作用域 Lexical scoping

不会从超类中继承任何变量名，也不会引入一个新的作用域。表达式函数体里边的变量和它的外部环境的变量具有相同的语义，this 关键字及其引用在表达式内部和外部也拥有相同的寓意

基于词法作用域的理念，lambda 表达式不可以掩盖任何其所在上下文中的局部变量，它的行为和那些拥有参数的控制结构一致（例如 for 循环）

## 变量捕获 Variable capture

如果一个局部变量在初始化后从未被修改过，那么它就符合有效只读的要求（加上`final`后也不会导致编译错误的变量就是有效只读变量）

## 方法引用 Method references

lambda 表达式允许我们定义一个匿名方法，并允许我们以函数式接口的方式使用它

```java
class Person {
  private final String name;
  private final int age;

  public int getAge() { return age; }
  public String getName() {return name; }
  ...
}

Person[] people = ...
Comparator<Person> byName = Comparator.comparing(p -> p.getName());
Arrays.sort(people, byName);

Comparator<Person> byName = Comparator.comparing(Person::getName);
```

通过名字是直接滴哦用。

- 静态方法引用 `ClassName::methodName`
- 实例方法引用 `instacneReference::methodName`
- 超类实例方法引用 `super::methodName`
- 类型上的实例方法引用 `ClassName::methodName`
- 构造方法引用 `Class::new`
- 数组构造方法引用 `TypeName[]::new`

## 默认方法和静态接口方法 Default and static interface methods

接口的方法可以是抽象的或是默认的，实现类通过继承得到该默认实现，通过重写来覆盖默认实现。

默认方法不是抽象方法，不用担心函数式接口的单抽象方法限制

静态方法可以定义在接口中，使得我们可以直接从接口调用和它相关的辅助方法

##　继承默认方法

当类型或者接口的超类拥有多个相同签名的方法时，需要一套规则来解决这个冲突

－　类的方法声明优先于接口默认方法，无论该方法是具体的还是抽象的
－　被其他类型所覆盖的方法会被忽略，适用于几个超类共享一个公共超类的情况

显式覆盖超类方法，一般来说会定义一个默认方法，然后在其中显式选择超类方法

```java
interface Robot implements Artist, Gun {
  default void draw() { Artist.super.draw(); }
}
```

## 示例

```java
List<Person> people = ...
Collections.sort(people, new Comparator<Person>() {
  public int compare(Person x, Person y) {
    return x.getLastName().compareTo(y.getLastName());
  }
})

Collections.sort(people,
                 (Person x, Person y) -> x.getLastName().compareTo(y.getLastName()));

Collections.sort(people, Comparator.comparing((Person p) -> p.getLastName()));

Collections.sort(people, comparing(p -> p.getLastName()));

Collections.sort(people, comparing(Person::getLastName));

people.sort(comparing(Person::getLastName));;

people.sort(comparing(Person::getLastName).reversed());;
```