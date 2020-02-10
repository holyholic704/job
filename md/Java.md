## 代码块

使用 `{}` 括起来的代码被称为代码块

- 在方法中出现
  - 普通代码块：限定变量生命周期，及早释放，提高内存利用率
  - 同步代码块：被 synchronized 修饰，语句块会自动被加上内置锁，从而实现同步

- 在类中方法外出现
  - 构造代码块：每次调用构造方法都会执行，并且 **在构造方法前执行**

  - 静态代码块：被 static 修饰，**在类被加载的时候执行**，**且只执行一次**，常用来加载驱动

### 执行顺序

父类静态代码块 -> 子类静态代码块 -> 父类普通代码块 -> 父类构造方法 -> 子类普通代码块 -> 子类构造方法

## 内部类

内部类就是在一个类的内部声明一个类，内部类 **不能定义静态变量和方法**。接口内也可以使用内部类，但没必要

### 内部类的作用

* 更好的实现隐藏：内部类可以被 private 与 protected 修饰
  
* **内部类拥有外部类的所有元素的访问权限**：可以直接调用所有外部类的成员，包括 private 修饰的变量和方法、非静态变量和方法等
  
* 可以间接的实现多重继承：**一个类中可以写多个内部类，分别继承不同的类**
  
* 可以避免实现的接口或继承的类中有同名的方法

### 成员内部类

定义在一个类的内部

```java
public class Test {
    public int i1 = 1;
    private static int i2 = 2;
    public void method(){ System.out.println("Test"); }

    class InnerClass{
        public void innerMethod(){
            System.out.println(i1);
            System.out.println(i2);
            method();
        }
    }
}
```

- 外部类想访问成员内部类的成员，必须 **先创建一个成员内部类的对象**
- 当成员内部类中拥有和外部类同名的成员变量或者方法时，**默认会调用成员内部类的成员**
- 如果要使用外部类的同名成员，可以通过 **`外部类名.this.成员变量/成员方法`** 调用

```java
// 其他类使用内部类，要先创建外部类的对象
public class Other{
    public static void main(String[] args){
        Test t =new Test();
        Test.InnerClass tic = t.new InnerClass();
        tic.innerMethod();
    }
}
```

### 局部内部类

定义在一个方法或者代码块里面的类，只能访问 **方法内或者代码块内的成员**。**不能被权限修饰符或 static 修饰**

```java
public class Test {
    public static void main(String[] args) {
        class InnerClass {
            int num = 1;
        }
        int n = new InnerClass().num;
        System.out.println(n);
    }
}
```

### 匿名内部类

没有名称的类，利用父类的构造函数和自身类体构造成一个类，其他地方不能引用，**没有构造器，不能实例化，只能使用一次**

```java
public class Test {
    public static void main(String[] args) {
        Test test = new Test();
        One one = new One() {
        	public void print() {
            	System.out.println("one");
        	}
    	};
        test.one.print();
        test.two.print();
    }
    Two two = new Two() {
        public void print() {
            System.out.println("two");
        }
    };
}
```

### 静态内部类

被 static 修饰的成员内部类，不需要依赖于外部类的，可以看做静态变量，不能使用外部类的非静态变量或方法

```java
public class Test {
    private static int num = 9;

    public static void main(String[] args) {
        Test.InnerClass.one();
        InnerClass ic = new Test.InnerClass();
        ic.two();
    }

    static class InnerClass {
        public static void one(){ System.out.println(num); }
        public void two(){ System.out.println(num); }
    }
}
```

## 异常

* Java 号称安全性，异常考虑到可能出现的错误，使用异常处理后把错误处理和真正的工作分开来
* 代码更易组织，更清晰，复杂的工作任务更容易实现
* 更安全，不至于由于一些小的疏忽而使程序意外崩溃了
* 在 Java 中，所有的异常都有一个共同的祖先 java.lang 包中的 **Throwable 类**
  * Throwable 有两个重要的子类：**Exception、Error**

### Error 与 Exception

#### Error

程序无法处理的错误，表示运行应用程序中较严重问题。大多数错误与代码编写者执行的操作无关，而表示代码运行时 JVM 出现的问题。这些异常发生时，JVM 一般会选择线程终止

- 这些错误是不可查的，因为它们在应用程序的控制和处理能力之外，而且绝大多数是程序运行时不允许出现的状况。对于设计合理的应用程序来说，即使确实发生了错误，本质上也不应该试图去处理它所引起的异常状况

#### Exception

程序本身可以处理的异常。Exception 类有一个重要的子类 RuntimeException。RuntimeException 异常由 Java 虚拟机抛出。异常和错误的区别在于，异常能被程序本身可以处理，错误是无法处理

### 可查异常与不可查异常

#### 可查异常（checked exceptions）

编译器要求必须处置的异常，即当程序中可能出现这类异常，要么用 try-catch 语句捕获它，要么用 throws 子句声明抛出它，否则编译不会通过。在正确的程序在运行中，很容易出现的、情理可容的异常状况。可查异常虽然是异常状况，但在一定程度上它的发生是可以预计的，而且一旦发生这种异常状况，就必须采取某种方式进行处理

- 除了 RuntimeException 及其子类以外，其他的都属于可查异常

#### 不可查的异常（unchecked exceptions）

编译器不要求强制处置的异常，一般是由程序逻辑错误引起的，在程序中可以选择捕获处理，也可以不处理

- 包括 RuntimeException 与其子类 和 Error

### 运行时异常与非运行时异常

#### 运行时异常

RuntimeException 类及其子类都是不可查异常，Java 编译器不会检查它，当程序中可能出现这类异常，即使没有用 try-catch 语句捕获它，也没有用 throws 子句声明抛出它，也可以编译通过。这些异常一般是由程序逻辑错误引起的，程序应该从逻辑角度尽可能避免这类异常的发生

|              异常              |       说明       |
| :----------------------------: | :--------------: |
|      ArithmeticException       |     算数异常     |
| ArrayIndexOutOfBoundsException | 数组下标越界异常 |
|       ClassCastException       |   类型转换异常   |
|     ClassNotFoundException     |   找不到类异常   |
|    IllegalArgumentException    |   非法参数异常   |
|     IllegalStateException      |   非法状态异常   |
|   IndexOutOfBoundsException    |   下标越界异常   |
|      NullPointerException      |    空指针异常    |
|     NumberFormatException      |   数字格式异常   |

#### 非运行时异常 （编译异常）

除 RuntimeException 类及其子类以外的异常。从程序语法角度讲是必须进行处理的异常，如果不处理，程序就不能编译通过

### Throwable 类常用方法

- `String getMessage()`：返回异常发生时的详细信息
- `String toString()`：返回异常发生时的简要描述
- `String getLocalizedMessage()`：返回异常对象的本地化信息。使用 Throwable 的子类覆盖这个方法，可以声称本地化信息
  - 如果子类没有覆盖该方法，则该方法返回的信息与 `getMessage()` 返回的结果相同
- `void printStackTrace()`：在控制台上打印 Throwable 对象封装的异常信息

### 异常处理

在 Java 应用程序中，异常处理机制为：**抛出异常，捕获异常**

#### 抛出异常

任何 Java 代码都可以抛出异常，当一个方法出现错误引发异常时，方法创建异常对象并交付运行时系统，异常对象中包含了异常类型和异常出现时的程序状态等异常信息，运行时系统负责寻找处置异常的代码并执行

#### 捕获异常

一个方法所能捕捉的异常，一定是 Java 代码在某处所抛出的异常，即异常总是 **先被抛出，后被捕捉**。在方法抛出异常之后，运行时系统将转为寻找合适的异常处理器。潜在的异常处理器是异常发生时依次存留在调用栈中的方法的集合

当异常处理器所能处理的异常类型与方法抛出的异常类型相符时，即为合适的异常处理器。运行时系统从发生异常的方法开始，依次回查调用栈中的方法，直至找到含有合适异常处理器的方法并执行。当运行时系统遍历调用栈而未找到合适的异常处理器，则运行时系统终止，同时，意味着 Java 程序的终止

### try-catch 与 finally

* try 块：用于捕获异常。其后可接 0 个或多个 catch 块，如果没有 catch 块，则必须跟一个 finally 块
* catch 块：用于处理 try 捕获到的异常
* finally 块：无论是否捕获或处理异常，finally 块里的语句都会被执行，除非以下四种特殊情况
  * 在 finally 语句块中发生了异常
  * 在前面的代码中用了 `System.exit()` 退出程序
  * 程序所在的线程死亡
  * 关闭 CPU

try 块里面的内容称为监控区域。Java 方法在运行过程中出现异常，则 **创建异常对象**。将异常抛出监控区域之外，由运行时系统寻找匹配的 catch 子句以捕获异常。若有匹配的 catch 子句，则运行其异常处理代码，try-catch 语句结束。如果抛出的异常对象属于 catch 子句的异常类或子类 ，则认为生成的异常对象与 catch 块捕获的异常类型相匹配

#### finally 和 return 的执行顺序

* try 和 catch 中有 return
  * finally 中的代码总会执行，而且 finally 语句在 return 语句执行之后，在 return 返回之前执行

* finally 中有 return
  * 会直接返回，不会再去返回 try 或者 catch 中的返回值

* finally 中对于返回变量做的改变是否会影响最终的返回结果
  * 如果 try 和 catch 的 return 是一个变量时且函数是从其中一个返回时，后面 finally 中语句即使有对返回的变量进行赋值的操作时，也不会影响返回的值

*更多：[finally 和 return的执行顺序](https://blog.csdn.net/jdfk423/article/details/80406297)*

### throw 与 throws

#### throw

throw 用在方法体中，用来抛出一个 Throwable 类型的异常。 程序会在 throw 语句后立即终止，它后面的语句不执行，然后在包含它的所有 try 块中，从里向外寻找含有与其匹配的 catch 子句的 try 块

如果抛出了检查异常 ，还应该在 **方法头部声明方法可能抛出的异常类型**，该方法的调用者也必须检查处理抛出的异常。如果所有方法都层层向上抛获取的异常，最终 JVM 会进行处理，打印异常消息和堆栈信息

#### throws

如果一个方法可能会出现异常，但没有能力处理这种异常，可以在方法声明处用 throws 子句来声明抛出异常。throws 语句用在方法定义时声明该方法要抛出的异常类型 ，多个异常可使用逗号分割。当方法抛出异常列表的异常时，方法将不对这些类型及其子类类型的异常作处理，而抛向调用该方法的方法或对象，由他去处理。还可以层层向上抛获取的异常，最终必须要有能够处理该异常的调用者

#### throw 和 throws 的区别

|                  throw                  |                  throws                   |
| :-------------------------------------: | :---------------------------------------: |
|   用在方法体内，跟的是 **异常对象名**   |   用在方法声明后面，跟的是 **异常类名**   |
|         只能抛出一个异常对象名          |      可以跟多个异常类名，用逗号隔开       |
| 表示抛出异常，由方法体内的 **语句处理** | 表示抛出异常，由该方法的 **调用者来处理** |

### 自定义异常类

```java
// 继承Exception类
public class MyException extends Exception{
    // 创建构造方法，调用父类构造方法
    public MyException(String str){
        super(str);
    }
}
```

### 异常链

将异常发生的原因一个传一个串起来，即把底层的异常信息传给上层，这样逐层抛出。当程序捕获到了一个底层异常，在处理部分选择了继续抛出一个更高级别的新异常给此方法的调用者。 这样异常的原因就会逐层传递。这样，位于高层的异常递归调用 getCause 方法，就可以遍历各层的异常原因

- 异常链的实际应用很少，发生异常时候逐层上抛不是个好注意，上层拿到这些异常又能怎么如何处理，而且异常逐层上抛会消耗大量资源，因为要保存一个完整的异常链信息

*更多：[Java异常](https://blog.csdn.net/qq_35101189/article/details/52938657)、[深入理解 Java中异常体系](https://blog.csdn.net/zhanaolu4821/article/details/81012382)*

## 泛型

把明确类型的工作推迟到创建对象或调用方法的时候，可以把运行时的问题提前到编译时期。**提供了编译期的类型安全**，确保把正确类型的对象放入集合中，避免了在运行时出现 ClassCastException 异常

* 可以明确集合中的数据类型，提高安全性、可读性，可以使用增强 for 循环遍历

- 代码更加简洁，不需要强制转换
- 程序更加健壮，只要编译时期没有警告，那么运行时期就不会出现 ClassCastException 异常

### 泛型的通配符

其实这些字母并没有实际用途，只有字面意思，只是一个约定的规范，`Test<T>` 与 `Test<A>` 在执行效果上并没有什么区别

* T：代表一个具体的 Java 类型
* E：代表 Element 的意思，或者 Exception 异常的意思
* K：代表 Key 的意思，通常与 V 一起配合使用
* V：代表 Value 的意思，通常与 K 一起配合使用
* S：代表 Subtype 的意思
* ?：**无限定的通配符**
* `? extends E`：**设定通配符上限**，接收 E 类型或者 E 的子类型
* `? super E`：**设定通配符下限**，接收 E 类型或者 E 的父类型

### 可以把 `List<String>` 传递给一个接受 `List<Object>` 参数的方法吗

会导致编译错误，因为 `List<Object>` 可以存储任何类型的对象包括 String、Integer 等，而 `List<String>` 却只能用来存储 String

### `Class<T>` 和 `Class<?>` 的区别

`Class<T>` 在实例化的时候，T 要替换成具体类，`Class<?>` 是个通配泛型，? 可以代表任何类型，主要用于声明时的限制情况

### PECS 原则

- 如果要从集合中读取类型 T 的数据，并且不能写入，可以使用 `? extends 通配符`（Producer Extends）
- 如果要从集合中写入类型 T 的数据，并且不需要读取，可以使用 `? super 通配符`（Consumer Super）
- 如果既要存又要取，那么就不要使用任何通配符

### 泛型的使用

#### 泛型类

泛型类型用于类的定义中，用户使用该类的时候，才把类型明确下来。通过泛型可以完成对一组类的操作对外开放相同的接口，如 List、Set、Map 等容器类。泛型类的子类，如果有明确的类型参数，需将用泛型的地方全部替换成传入的实参类型，否则需将泛型的声明也一起加到类中。**类上声明的泛型只对非静态成员有效**，如果静态方法要使用泛型的话，**必须将静态方法也定义成泛型方法**

```java
public class Test<T> {
    private T obj;
    
    public T getObj() {
        return obj;
    }
    
    public void setObj(T obj) {
        this.obj = obj;
    }
    
    public static void main(String[] args) {
        // 在实例化泛型类时，必须指定T的具体类型
        Test<Integer> test = new Test<>();
        test.setObj(110);
        System.out.println(test.getObj());
    }
}
```

#### 泛型接口

- 与泛型类相似

```java
public interface Test<T> {
    T anything();
}

// 当实现泛型接口的类，未传入泛型实参时，在声明类的时候，需将泛型的声明也一起加到类中
// 如果不声明泛型，编译器会报错
public class One<T> implements Test<T> {
    T anything();
}

// 当实现泛型接口的类，传入泛型实参时，需将用泛型的地方全部替换成传入的实参类型
public class One<String> implements Test<String> {
    String anything();
}
```

#### 泛型方法

在调用方法的时候指明泛型的具体类型，类型参数写在返回值前面，声明的类型参数，还可以当作返回值的类型

```java
public class Test {
    // 只有声明了<T>的方法才是泛型方法，泛型类中的使用了泛型的成员方法并不是泛型方法
	public <T> T method(T t){
        return t;
    }
    
    public static void main(String[] args) {
        Test test = new Test();
        System.out.println(test.method("fuck"));
        System.out.println(test.method(12));
    }
}
```

### 泛型擦除

泛型是通过类型擦除来实现的，泛型信息只存在于代码编译阶段，在进入 JVM 之前，与泛型相关的信息会被擦除

#### 局限性

泛型擦除，是泛型能够与之前的 Java 版本代码兼容共存的原因。但也因为类型擦除，它会抹掉很多继承相关的特性，这是它带来的局限性

```java
List<Integer> list = new ArrayList<>();
list.add(1);
// 因为泛型的限制，以下代码会报错，基于对类型擦除的了解，利用反射，就可以绕过这个限制
// list.add("String");list.add(27.12);会报错
Method method = list.getClass().getDeclaredMethod("add", Object.class);
method.invoke(list, "String");
method.invoke(list, 27.12);

for (Object o : list) {
    System.out.println(o);	// 成功打印
}
```

### 泛型数组

Java 不能创建具体类型的泛型数组

```java
// 这两行代码是无法在编译器中编译通过的。原因是类型擦除带来的影响
List<Integer>[] list = new ArrayList<Integer>[];
List<Boolean> list = new ArrayList<Boolean>[];
```

`List<Integer>` 和 `List<Boolean>` 在 JVM 中等同于 `List<Object>`，所有的类型信息都被擦除，程序也无法分辨一个数组中的元素类型具体是 `List<Integer>` 类型还是 `List<Boolean>` 类型

```java
List<?>[] list = new ArrayList<?>[10];
list[1] = new ArrayList<String>();
List<?> v = list[1];
```

借助于无限定通配符却可以，？代表未知类型，所以它涉及的操作都基本上与类型无关，因此 JVM 不需要针对它对类型作判断，因此它能编译通过

*更多：[泛型就这么简单](https://segmentfault.com/a/1190000014120746)、[java 泛型详解](https://blog.csdn.net/s10461/article/details/53941091)、[Java 泛型，你了解类型擦除吗](https://blog.csdn.net/briblue/article/details/76736356)、[JAVA泛型通配符T，E，K，V区别](https://www.jianshu.com/p/95f349258afb)*

