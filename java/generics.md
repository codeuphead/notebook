# 泛型

JDK 1.5 引入泛型，泛型提供了编译时类型安全检测机制，该机制允许在编译时检测到非法类型。

其本质是将类型参数化，即所操作的数据类型被指定为一个参数，是不确定的。

- 使用泛型可以编写模板代码来处理任何类型，减少重复代码
- 使用泛型时不必对类型进行强制转换，避免转型类型错误

## 泛型使用

### 泛型类

实例化时指明泛型类型的具体参数

- 类名后加一个尖括号，括号里写泛型标识，作为泛型的声明
- 泛型标识可以有多个，用逗号隔开
- 泛型类，如果没有指定具体的数据类型，使用 Object
- 泛型的类型参数只能是引用类型，不能是基本类型
- 子类是泛型类，则子类和父类的泛型类型要一致，子类可以增加泛型类型数量
- 子类不是泛型类，则要明确父类的泛型的数据类型

```java
class Point<T, E> {
    private T x;
    private E y;

    public void setX(T x){
        this.x = x;
    }

    public T getX(){
        return this.x;
      }


    public void setY(E y){
        this.y = y; 
    }

    public E getY(){
        return this.y;
    }
}

Point<Integer, Integer> p = new Point<>();
p.setX(100);
p.setY(200);

Point<String, String> p2 = new Point<>();
p2.setX("100");
p2.setY("200");
```

### 泛型接口

类似泛型类

```java
interface Point<T, E> {
    T getX();
    E getY();
}
```

### 泛型方法

调用方法时，指明泛型类型的具体类型

- 在返回值前声明泛型标识
- 返回值也可以替换成泛型
- 与泛型类一样可以声明多个

```java
public class Hello {

    public <T> T printArray(T[] srcArr){
        for(E el : srcArr){
            System.out.println(el.toString());
        }

        return src[arr][0];
    }

    public static void main(String args[]){
        Integer[] intArray = {1, 2, 3, 4, 5};
        String[] stringArray = {"1", "2", "3", "4", "5"};
        printArray(intArray);
        printArray(stringArray);
    }
}
```

- 泛型类中，使用了类声明的泛型的方法不是泛型方法，在返回值前声明了泛型标识的才是
- 泛型类中，如果有泛型方法，即使他们两者的泛型标识名称一样，但他们也不是一个东西，互不干扰。(可以看作是独立的，或作用域在方法的类型参数)
- 使用了类泛型的方法不能是静态方法，但是泛型方法可以是静态

```java
public class Hello<T> {

    private T name;

    public T getName(){
        return name;
    }

    public <T> T printArray(T[] srcArr){
        for(E el : srcArr){
            System.out.println(el.toString());
        }

        return src[arr][0];
    }

}
```

## 通配符

当使用的变量含有泛型类型时，无法通过一个声明匹配多种实际类型，因此需要使用通配符。

### 无边界通配符 `?`

用于填充类型参数，是类型实参，而不是类型形参

```java
class Point<T> {
    private T x;
    private T y;

    public Point(T x, T y){
        this.x = x;
        this.y = y;
    }

    public String showPoint(){
        System.out.println("Point " + x + ":" + y);
    }

    public static void main(String args[]) {
        Point<?> point;
        point = new Point<Integer>(3,3);
        doShowPoint(point);
        point = new Point<Float>(4.3F,4.3F);
        doShowPoint(point);
        point = new Point<Double>(4.3D,4.90D);
        doShowPoint(point);
        point = new Point<Long>(12L,23l);
        doShowPoint(point);

    }

    public static void doShowPoint(Point<?> point) {
        point.showPoint();
    }
}

```

### 通配符上限

- 通配符的 extends 绑定，表示填充的类型上限为 xxx，可以填充 xxx 子类
  
```java
Point<? extends Number> point;
point = new Point<Integer>(3,3);
point = new Point<Float>(4.3f,4.3f);
point = new Point<Double>(4.3d,4.90d);
point = new Point<Long>(12l,23l);
```

```java
Point<? extends Number> point;
point = new Point<Float>();
point = new Point<Number>();
point = new Point<String>(); //错误 因为 String 不是 Number 的子类
```

- 只能取值，不能添加

```java
Point<? extends Number> point;
point = new Point<Integer>(2);
Number n = point.getX();
point.setX(new Integer(222)); //错误
```

通配符 `<? extends Number>` 表示某种特定类型，但是不关心实际类型是什么，此时 `setX(T a)` 就变成了 `setX(? extends Number)`

point 的泛型变量 T 类型是 `<? extends Number>`，Number 是它的上边界，但是我们不知道其持有的具体类型，是未知的，因此无法 set 值，唯一可以添加的是 null，但是没有意义

如果返回 T 类型，虽然我们不知道实际类型，但是我们知道它肯定是 Number 的子类，因此肯定能转型成 Number

### 通配符下限

- 通配符 super 绑定，表示填充为 xxx 的父类

```java
class CEO extends Manager {
}
class Manager extends Employee {
}
class Employee {
}
```

```java
List<? super Manager> list; // 将 T 填充为任意 Manager 的父类
list = new ArrayList<Employee>();
list = new ArrayList<Manager>();
list = new ArrayList<CEO>(); //编译错误，因为 CEO 不是 Manager 的父类
```

- 只能添加，不能取值

```java
List<? super Manager> list = new ArrayList<Employee>();
//存
list.add(new Employee()); //编译错误
list.add(new Manager());
list.add(new CEO());

Employee item = list.get(0); //错误
```

list 类型是由 `List<? super Manger>`决定的，即 Manager 的任意父类，但是是哪一个父类，并不能确定，因此 add Employee 是错误的，因为不能确定 Employee 是 `<? super Manger>` 的父类

list 取值的时候，item 的类型是 `<? super Manger>`，遍历时使用的是 Object

### 总结

- PECS
  
Producter - Extend - Consumer - Super

- 如果你想从一个数据类型里获取数据，使用 `? extends` 通配符（能取不能存）
- 如果你想把对象写入一个数据结构里，使用 `? super` 通配符（能存不能取）
- 如果你既想存，又想取，那就别用通配符。
- 构造泛型实例时，如果省略了填充类型，则默认填充为无边界通配符


## 类型擦除

Java 的泛型只存在于编译期，编译后泛型会被擦除，可以从编译后的字节码文件查看。

- 泛型类擦除后，是类的原始的类型。
- 泛型类中，对于使用了了泛型的变量，其擦除类型后，是 Object；如果指定了泛型上限，则编译后的类型是上限类型
- 泛型方法中，类似。
- 桥接方法：如果接口泛型，擦除后变成 Object，但是**实现类**擦除后，会生成一个桥接方法，重写接口中的方法，来保持类的实现，同时也会有一个同名的方法，其泛型擦除后，与传入的类型一致

```java
public class ErasedTypeEquivalence {
    public static void main(String[] args) {
        Class c1 = new ArrayList<String>().getClass();
        Class c2 = new ArrayList<Integer>().getClass();
        System.out.println(c1.getSimpleName());
        System.out.println(c2.getSimpleName());
        System.out.println(c1 == c2);
    }
}
```

- 两个 List 的类型是 ArrayList
- 两个 List 的参数类型不一样，运行后代码输出是 true，说明 JVM 把他们当作同一个类型，而在 C++ C# 中，他们就是不同的类型
- 泛型信息被擦除，但是在 class 文件中以 Signature 形式保留在 Constant pool 中，通过 javap 命令可查看，方法的局部变量的泛型信息是不会被保存的

### 创建对象

```java
public class Erased<T> {
    private final int SIZE = 100;
    public static void f(Object arg) {
    if(arg instanceof T) {} // Error
    T var = new T(); // Error
    T[] array = new T[SIZE]; // Error
    T[] array = (T)new Object[SIZE]; // Unchecked warning
    }
}
```

任何在运行期需要知道确切类型的代码都无法工作，如上，无法创建对象。

可以使用工厂或者模板方法来生成对象。

## 泛型数组

Java 中不允许直接创建泛型数组，可以声明。

`T[]`

由于擦除，运行期的数组类型只能是 Object[]，如果我们立即把它转型为 T[]，那么在编译期就失去了数组的实际类型，编译器也许无法发现潜在的错误。

因此，更好的办法是在内部最好使用 Object[] 数组，在取出元素的时候再转型。看下面的例子：

```java
public class GenericArray2<T> {
    private Object[] array;
    public GenericArray2(int sz) {
        array = new Object[sz];
    }
    public void put(int index, T item) {
        array[index] = item;
    }

    public T get(int index) { return (T)array[index]; }

    public T[] rep() {
        return (T[])array; // Warning: unchecked cast
    }
    public static void main(String[] args) {
        GenericArray2<Integer> gai =
        new GenericArray2<Integer>(10);
        for(int i = 0; i < 10; i ++)
        gai.put(i, i);
        for(int i = 0; i < 10; i ++)
        System.out.print(gai.get(i) + " ");
        System.out.println();
        try {
            Integer[] ia = gai.rep();
        } catch(Exception e) { System.out.println(e); }
    }
}
```

### 类型标识

使用 Class 对象作为类型的标识是更好的设计

```java
public class GenericArrayWithTypeToken<T> {
    private T[] array;

    public GenericArrayWithTypeToken(Class<T> type, int sz) {
        array = (T[])Array.newInstance(type, sz);
    }
    public void put(int index, T item) {
        array[index] = item;
    }
    public T get(int index) { return array[index]; }
    public T[] rep() { return array; }
    public static void main(String[] args) {
        GenericArrayWithTypeToken<Integer> gai =
            new GenericArrayWithTypeToken<Integer>(Integer.class, 10);
        Integer[] ia = gai.rep();
    }
}
```

在构造器中传入了 `Class<T>` 对象，通过 `Array.newInstance(type, sz)` 创建一个数组，这个方法会用参数中的 Class 对象作为数组元素的组件类型。

这样创建出的数组的元素类型便不再是 Object，而是 T。

这个方法返回 Object 对象，需要把它转型为数组。不过其他操作都不需要转型了，包括 rep() 方法，因为数组的实际类型与 T[] 是一致的。

这是比较推荐的创建泛型数组的方法。

