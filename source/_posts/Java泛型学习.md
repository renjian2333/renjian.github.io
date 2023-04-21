---
title: Java泛型学习
toc: true
date: 2023-04-11 15:48:54
tags: Java
categories: Java基础学习
---
参考资料：
>  [韩顺平零基础30天学Java](https://www.bilibili.com/video/BV1fh411y7R8?p=555)
> 《Java核心技术 卷I》

## 泛型的引入
* 不能对加入到集合中的数据类型进行约束(不安全)
* 集合遍历的时候需要进行类型转换，对效率有影响
> 使用泛型机制编写代码比那些杂乱地使用Object变量，然后再进行强制类型转换的代码具有更好的安全性和可读性。泛型对于集合类尤其有用。

### 泛型介绍
* 泛型又称参数化类型，是Jdk5.0出现的新特性，解决数据类型的安全性问题。(泛型类可看作普通类的工厂)
* Java泛型可以保证如果程序在编译时没有发出警告，运行时就不会产生ClassCastException异常。同时，代码更简洁、健壮。
* 在类声明时通过一个标识表示类中某个属性的类型，或者是某个方法返回值的类型，或者是参数类型。
* 在实例化时要指定好需要的具体类型。

### 使用注意事项
* 给泛型指定数据类型要求是引用类型，**不能是基本数据类型**
* 在泛型指定后，可以传入该类型及其子类类型
* 泛型默认是Object
  ```Java
  ArrayList list = new ArrayList(); //等价于下面
  ArrayList<Object> list = new ArrayList<>();
  ```
* 类型变量通常使用大写字母表示。在Java 库中，使用变量E表示集合的元素类型，K和V分别表示表的关键字与值的类型。T(需要时还可以用临近的字母U和S)表示“任意类型”。

## 自定义泛型
### 泛型类
* 基本语法
  ```Java
  class ClassName<T,E>{ //可以有多个泛型
    T field1;
    E field2;

    public T func1() {

    }
  }
  ```
* 普通成员可以使用泛型(属性、方法)
* 使用泛型的数组不能初始化(因为数组在new的时候不能确定泛型的类型，无法开辟对应的内存空间)
* 静态方法和属性不能使用泛型(静态方法和类相关，在类加载时，对象还没创建，JVM无法完成初始化)

### 泛型接口
* 基本语法
  ```Java
  interface Name<T,R>{
    void run(T t);

    // default 用来修饰接口中有实现的方法
    default R method(T t){
        return null;
    }
  }
  ```
* 泛型接口的类型在继承接口或者实现接口的时候确定
* 静态成员不能使用泛型

### 泛型方法
* 基本语法
  ```Java
  // 类型变量放在修饰符之后，返回变量之前
  class ArrayAlg{
    public static <T> T getMiddle(T... a){
      return a[a.length / 2];
    }
  }
  ```
* 泛型方法可以定义在普通类中，也可以定义在泛型类中
* 调用方法时传入参数编译器会自动确定类型
  ```Java
  String middle = ArrayAlg.<String>getMiddle("John","Wang","ss"); //尖括号通常省略
  ```
* 泛型方法和使用泛型的方法不一样
  ```Java
  public<T> void func(T t){}
  public void func(T t){}
  ```

## 类型变量的限定
类或方法有时需要对类型变量加以约束, 举例说明:
```Java
class ArrayAlg {
  public static <T extends Comparable> T min(T[] a){
    if(a == null || a.length == 0) return null;
    T smallest = a[0];
    for(int i = 1; i < a.length; i++){
      if(smallest.compareTo(a[i] > 0)) smallest = a[i];
    }
  }
}
```
为了保证内部逻辑正确运行，需要保证传入的T的类型必须实现了Comparable接口，否则会产生编译错误。
* 限定类型用“&” 分隔，而逗号用来分隔类型变量。
  ```Java
  class A <T extends Serializable & Cloneable>{}
  class B <T extends Serializable, E>{}
  ```
* 可以根据需要拥有**多个接口**超类型，但限定中**至多有一个类**。如果用一个类作为限定，它必须是限定列表中的第一个。
  ```Java
  class C <T extends Cloneable & A>{} // 错误
  class C <T extends A & Cloneable>{} // 正确
  ```

## 泛型的继承和通配符
* 泛型不具备继承性，但泛型类可以拓展或实现其他的泛型类
  ```Java
  List<Object> list = new ArrayList<String>(); // 错误写法
  public class ArrayList<E> extends AbstractList<E> implements List<E>{} //ok
  ```
* <\?>: 支持任意类型
* <\? extends A>: 支持A类及A类的子类
* <\? super A>: 支持A类及A类的父类
  ```Java
  public class Test<T> {
    public static void main(String[] args) {
        List<Object> list1 = new ArrayList<>();
        List<AA> list2 = new ArrayList<>();
        List<BB> list3 = new ArrayList<>();
        List<CC> list4 = new ArrayList<>();
        
        printCollection1(list1);
        printCollection1(list2);
        printCollection1(list3);
        printCollection1(list4);
        
        printCollection2(list1); // 编译报错
        printCollection2(list2); // 编译报错
        printCollection2(list3);
        printCollection2(list4);

        printCollection3(list1);
        printCollection3(list2);
        printCollection3(list3);
        printCollection3(list4); // 编译报错
    }

    public static void printCollection1(List<?> c){
        for (Object o :c) {
            System.out.println(o);
        }
    }

    public static void printCollection2(List<? extends BB> c){
        for (Object o :c) {
            System.out.println(o);
        }
    }

    public static void printCollection3(List<? super BB> c){
        for (Object o :c) {
            System.out.println(o);
        }
    }
  }

  class AA {}
  class BB extends AA {}
  class CC extends BB {}
  ```
* 带有超类型限定的通配符可以向泛型对象写入，带有子类型限定的通配符可以从泛型对象读取。

## 类型擦除
* 虚拟机中没有泛型，只有普通的类和方法。
* 无论何时定义一个泛型类型，都自动提供了一个相应的原始类型(raw type)。原始类型的名字就是删去类型参数后的泛型类型名。擦除(erased) 类型变量, 并替换为(第一个)限定类型(无限定的变量用Object)。
* 切换限定：class Interval<T extends Serializable & Comparable>会发生什么？如果这样做，原始类型用Serializable 替换T, 而编译器在必要时要向Comparable 插入强制类型转换。为了提高效率，应该将标签(tagging) 接口(即没有方法的接口)放在边界列表的末尾。
* 当程序调用泛型方法时，如果擦除返回类型，编译器插入强制类型转换。
* 泛型方法的类型擦除可能会导致与多态性的冲突，需要借助桥方法(bridge method)解决冲突。

## 约束与局限性
* 不能用基本类型实例化类型参数(因为类型擦除)
* 运行时类型查询只适用于原始类型
  * 不能使用instanceof判断是否是泛型类型
  * getClass总是返回原始类型
* 不能创建参数化类型的数组(类型擦除导致，但可以声明参数化类型数组)；如果需要收集参数化类型对象，只有一种安全而有效的方法：使用ArrayList:ArrayList<ClassName<String>>
* Varargs警告
* 不能实例化类型变量(如：new T())
* 不能构造泛型数组(如：T[])
* 泛型类的静态上下文中类型变量无效, 即不能在静态域或方法中引用类型变量。
