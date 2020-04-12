# Effective Java

## 一. 创建和销毁对象

### 1. 静态工厂方法代替构造器

优点：

1. 有方法名称，从方法名称上就可以区分创建不同对象
2. 不必再每次调用它们的时候都创建一个对象
3. 可以返回原返回类型的任何子类型的对象
4. 创建参数化实例的时候，可以使代码更加简洁

缺点

1. 类如果不含有共有的或者受保护的构造器，就不能被子类化

2. 与静态方法实际上没有区别

   常用命名：

   - valueOf 返回的实例与它的参数具有相同的值
   - of valueOf的一种更为简洁的写法
   - getInstance 返回的实例通过方法的参数来描述
   - newInstance 像getInstance一样，但newInstance能确保返回的每个实例都与其他实例不同
   - getType 像getInstance一样，在工厂方法处于不同的类中使用，Type表示工厂方法返回的类型
   - newType 像getInstance一样，在工厂方法处于不同的类中使用，Type表示工厂方法返回的类型 

### 2. 遇到多个构造器参数使用构建器

使用Builder模式既可以保证像重叠构造器模式那样的安全性，同时也能保证像JavaBean模式那样那么好的可读性。

```java
public class User {
    private final String firstName;     // 必传参数
    private final String lastName;      // 必传参数
    private final int age;              // 可选参数
    private final String phone;         // 可选参数
    private final String address;       // 可选参数

    private User(UserBuilder builder) {
        this.firstName = builder.firstName;
        this.lastName = builder.lastName;
        this.age = builder.age;
        this.phone = builder.phone;
        this.address = builder.address;
    }

    public String getFirstName() {
        return firstName;
    }

    public String getLastName() {
        return lastName;
    }

    public int getAge() {
        return age;
    }

    public String getPhone() {
        return phone;
    }

    public String getAddress() {
        return address;
    }

    public static class UserBuilder {
        private final String firstName;
        private final String lastName;
        private int age;
        private String phone;
        private String address;

        public UserBuilder(String firstName, String lastName) {
            this.firstName = firstName;
            this.lastName = lastName;
        }

        public UserBuilder age(int age) {
            this.age = age;
            return this;
        }

        public UserBuilder phone(String phone) {
            this.phone = phone;
            return this;
        }

        public UserBuilder address(String address) {
            this.address = address;
            return this;
        }

        public User build() {
            return new User(this);
        }
    }
}
```

注意：

1. User类的构造方法是私有的。也就是说调用者不能直接创建User对象。

2. ****User****类的属性都是不可变的。所有的属性都添加了final修饰符，并且在构造方法中设置了值。并且，对外只提供`getters`方法。

3. Builder模式使用了链式调用。可读性更佳。

4. Builder的内部类构造方法中只接收必传的参数，并且该必传的参数适用了final修饰符。

### 3. 使用私有构造器或枚举类型强化Singleton属性

实现单例方式有三种：

1. 饿汉式

```java
public class Elvis{
    public static final Elvis INSTANCE = new Elvis();
    private Elvis(){……}
    ……
}
```

2. 懒汉式

```java
public class Elvis{
    private static final Elvis INSTANCE = new Elvis();
    private Elvis(){……};
    public static Elvis getInstance() {return INSTANCE};
    ……
}
```

3. Enum

```java
public class Elvis{
    INSTANCE;
    ……
}
```

这种方式更加简洁，无偿提供了序列化的能力，防止多次实例化，是实现Singleton的最佳方法。

### 4. 通过私有构造器强化不可实例化的能力

### 5.  避免创建不必要的对象

一般来说，最好能重用对象而不是每次需要的时候创建一个出来，例如

```java
String s = new String("hello")
```

每次被执行都会创建一个新的String实例，当在一个循环或者频繁被调用时，就会创建出许多不必要的实例。

同时当心自动装箱拆箱所带来的性能影响。要优先使用基本类型而不是装箱类型。

### 6. 消除过期的引用

1. 消除过期引用的好处：减少内存泄漏的风险

2. 内存泄漏的情形主要包括三种情况：

   1. 只要类是自己管理内存，就应该警惕内存泄漏的风险

      - 例子：Stack出栈时，出栈元素引用未被置为Null，栈一直保留着该元素的引用，导致内存泄漏

      解决办法：Stack栈出栈时将出栈元素引用置为Null。

   2. 缓存

      - 例子：将对象引用放在缓存中，被遗忘。

      - 解决办法：使用WeakHashMap代表缓存，或者缓存中项过期后自动被删除LinkedHashMap使用removeEldestEntry方法清理缓存

   3. 监听器和其他回调

      - 例子：在API中注册回调，却没有显式取消注册，导致内存泄漏

      - 解决办法：只保存它们的弱引用

### 7. 避免使用终结方法

原因：终结方法通常是不可预测的，也是危险的，一般情况下是不必要的。使用终结方法会导致行为不稳定、降低性能，以及可移植性的问题。应该避免使用终结方法。

终结方法（finalizer）缺点：

1. 不能保证会被及时地执行
2. 终结方法有非常严重的性能损失

## 三. 对于所有对象都通用的方法

### 1. 覆盖equals时请遵守通用约定

实现高质量equals方法诀窍：

1. 使用==操作符检查"参数是否为这个对象的引用"。
2. 使用instanceof操作符检查"参数类型是否正确"
3. 把参数转换成正确的类型
4. 对于该类中的每个"关键（significant）"域，检查参数中的域是否与该对象中对应的域相匹配
5. 当你编写完成了equals方法之后，应该问自己三个问题：它是否是对称的、传递的、一致的

告诫：

```
1. 覆盖equals时总要覆盖hashCode
```
	- 不要企图让equals方法过于智能
	- 不要将equals声明中的Object对象替换为其他的类型

### 2. 覆盖equals时总要覆盖hashCode

若不这样做，就会违反Object.hashCode的通用约定，从而导致该类无法结合所有基于散列的集合一起正常工作

### 3. 始终要覆盖toString

虽然遵守toString的约定并不像equals和hashCode的约定那么重要，但是提供好的toString实现可以使类用起来更加舒适，并且会产生有用的诊断信息。

### 4. 谨慎覆盖clone

覆盖clone方法的坏处

	1. 如果对象中包含的域引用了可变的对象，简单地使用clone方法可能会导致clone出来的对象会伤害到原始的对象
 2. clone架构与引用可变对象的final域的正常用法是不相兼容的
 3. clone散列表等数据时需要避免伤害到原始对象

其他接口都应该扩展这个接口，为了设计而继承的类也不应该实现这个接口

### 5. 考虑实现comparable接口

一旦类实现了Comparable接口，就可以和许多泛型算法进行协作以及依赖于该接口的集合实现进行协作，如果正则编写一个值类，它具有非常明确的内在排序关系，就应该坚决考虑实现这个接口。

## 四. 类和接口

### 1. 使类和成员的可访问性最小化

信息隐藏，软件设计的基本原则之一，设计良好的模块会隐藏所有的实现细节，把它的API与它的实现清晰地隔离开来，然后模块之间只通过它们的API进行通信，一个模块不需要知道其他模块的内部实现细节。

规则很简单：尽可能地使每个类或者成员不被外界访问

| private         | 私有     | 类内部访问                 |
| --------------- | -------- | -------------------------- |
| default（缺省） | 包级私有 | 包内部可访问               |
| protected       | 受保护   | 子类可访问、包内部也可访问 |
| public          | 公有     | 任何地方都可以访问         |

### 2. 在公有类中使用访问方法而非公有域

比如一些实例类

```java
Class Point{
    public double x;
    public double y;
}
```

该Point类的数据域是可以直接访问的，当数据被访问的时候，无法采取任何辅助的行动。所以应该采用getter()和setter()的类进行代替。

如果类可以在它所在的包的外部进行访问，就提供访问方法；如果类是包级私有的嵌套类，直接暴露它的数据域并没有本质的错误。

### 3. 使可变性最小化

不可变类只是其实例不能被修改的类。每个实例中包含的所有信息都必须在创建该实例的时候就提供，并在对象的整个生命周期内固定不变。不可变的类比可变类更加易于设计、实现和使用。它们不容易出错且更加安全，线程安全，唯一的缺点是可能存在性能问题。

使类成为不可变的规则：

	1. 不要提供任何会修改对象的方法
 	2. 保证类不会被扩展，防止子类化，使类成为final的，或者使用静态工厂方法和私有构造器结合
 	3. 使所有的域都是final的
 	4. 使所有的域都成为私有的
 	5. 确保对于任何可变组件的互斥访问

### 4. 复合优先于继承

继承是实现代码重用的有力手段，但并非最佳工具，使用不当会导致软件变得脆弱，再包的内部使用继承使非常安全的。但继承打破了封装性，换句话说，子类依赖于其超类中特定功能的实现细节，超类的实现有可能会随着发行版本的不同而有所变化，子类就可能会遭到破坏。因而子类必须要跟着超类的更新而演变。

幸运的是，有一种办法可以避免出现这种情况，不用扩展现有的类，而是在新的的类中增加一个私有域，它引用现有类的一个实例。这种设计叫做复合。

新类中的每个实例方法都可以调用被包含的现有类实例中的对应方法，并返回它的结果。这称为转发。这样的类不依赖现有类的实现细节，非常稳固。

包装类几乎没有什么缺点，需要注意的一点是包装类不适合在回调框架中。

只有当子类真正是超类的子类型时，才适合用继承。换句话说，对于两个类A、B，只有当两者之间确实存在is-a的关系时，类B才应该扩展类A。如果不确定时is-a的关系，那么B就不应该扩展A。如果答案时否定的，通常情况下，B应该包含A的一个私有实例，并暴露一个较小、简单的API：A本质上不是B的一部分，只是它的实现细节。

### 5. 要么为继承而设计，并提供文档说明，要么就禁止继承

对于不是为了继承而设计、并且没有文档说明的类进行子类化是危险的。该类的文档必须精确地描述覆盖每个方法所带来的影响。同时对于为了继承而设计的类，唯一的测试方法就是编写子类。也必须遵守一些约束：构造器绝不能调用可被覆盖的方法。

所以最佳实践是对于那些并非为了安全地进行子类化而设计和编写文档的类，要禁止子类化。

### 6. 接口优先于抽象类

Java提供了两种机制用来允许多个实现的类型：接口和抽象类。区别在于：抽象类允许包含某些方法的实现、但是接口则不允许。但对于抽象类，为了实现由抽象类定义的的类型、类必须成为抽象类的一个子类。

现有的类可以很容易地被更新、以实现新的接口。

接口是定义mixin（混合类型）的理想选择。

接口允许我们构造非层次结构的类型框架。

通过对导出的每个重要接口都提供一个抽象的骨架实现（skeletal implementtation）类，把接口和抽象类的有点结合起来。

骨架实现的美妙之处在于，它们为抽象类提供了实现上的帮助，但又不强加“抽象类用作类型定义”所特有的严格限制。此外骨架实现类仍然能够有助于接口的实现。实现了这个接口的类可以把对于这个接口方法的调用，转发到一个内部私有类的实例上，这个内部私有类扩展了骨架实现类。这种方法被成为多重继承

~~~java
public abstract class AbstractMapEntry<K,V> implements Map.Entry<K,V>{
    public abstract K getKey();
    public abstract V getValue();
    
    public V setValue(V value){
        throw new UnsupportedOperationException();
    }
    ……
}
~~~

使用抽象类类定义允许多个实现的类型，与使用接口相比有一个明显的优势：抽象类的演变比接口的演变要容易得多。

### 7. 接口只用于定义类型

当类实现接口时，接口就充当可以引用这个类的实例的类型。表明客户端可以对这个类的实例实施某些动作。为了其他目的而定义接口都是不恰当的。

有一种接口被称为常量接口，他只包含静态的final域，每个域都导出一个常量

~~~java
public interface PhysicalConstants{
    static final double NUM = 1.0;
    static final double PI = 3.14;
}
~~~

常量接口是对接口的不良使用。实现常量接口，会导致把这样的实现细节泄漏到该类的API中；如果将来的发行版中，这个类被修改了，它不再需要使用这些常量了，它依然必须实现这个接口，以确保兼容性。如果非final的类实现了常量接口，它的所有子类也会被接口中的常量所“污染”。

如果这些常量域某个现有的类或者接口紧密相关，就应该把这些常量添加到这个类或者接口中。

~~~java
public class PhysicalConstants{
    private PhysicalConstants(){}
    static final double NUM = 1.0;
    static final double PI = 3.14;
}
~~~

同时可以使用静态导入机制讲常量导入类中

~~~java
import static com.hu.PhysicalConstants.*
~~~

简而言之，接口应该只被用来定义类型，而不应该被用来导出常量。

### 8. 类层次优于标签类

有时候可能会遇到两种甚至多种风格的实例的类

~~~java
class Figure{
    enum Shape{RETANGLE,CIRCLE};
    final Shape shape;
    //RETANGLE
    double length;
    double width;
    //CIRCLE
    double radius;
    Figure(double radius){
        //init for CIRCLE
    }
    Figure(double length;double width){
        //init for RETANGLE
    }
}
~~~

这种标签类过于冗长、容易出错、并且效率低下。

将其子类化：

~~~java
abstract class FIgure{
    abstract double area();
}
class Circle extends Figure{
    final double radius;
    ……
}
class Rectangle extends Figure{
    final double length;
    final double width;
    ……
}
~~~

使用类层次后的优点：

	1. 代码简单清晰，无冗余代码，无样板代码
 	2. 每个类型的实现都有自己的类，没有受到不相关数据的拖累
 	3. 所有域都是final的，编译器确保每个类的构造器都初始化数据域
 	4. 可以独立扩展层次结构
 	5. 反映类型之间本质上的层次关系，有助于增强灵活性。

### 9. 用函数对象标识策略

Java没有提供函数指针，但是可以用对象引用实现同样的功能。对象的方法执行其他对象上的操作

~~~java
class StringLengthComparator{
    public int compare(String s1,String s2){
        return s1.length()-s2.length();
    }
}
~~~

作为具体策略类，StringLengthComparator类是无状态的，没有域，因此作为一个Singleton是非常合适的，可以节省不必要的对象创建开销。

此外可以创建一个Comparator接口用来扩展策略

~~~java
public interface Comparator<T>{
    public int compare(T t1,T t2);
}
~~~

此外，具体的策略类可以是宿主类的私有嵌套类

简而言之，函数指针的主要用途就是实现策略模式。为了在Java中实现这种模式，需要声明一个接口来表示该策略，并且为每个具体的策略声明一个实现了该接口的类。当一个具体的策略只被使用一次时，通常使用匿名类来声明和实例化这个具体策略类。当一个具体的策略是设计用来重复使用的时候，它的类通常就要被实现为私有的静态成员类，并通过公有的静态final域被导出，其类型为该策略接口。

### 10. 优先考虑静态成员类

嵌套类有四种：静态成员类、非静态成员类、匿名类、局部类。

1. 静态成员类是最简单的一种嵌套类，最好把它看作是普通的类，只是碰巧被声明在另一个类的内部而已，静态成员类的一种常见用法是作为公有的辅助类。
2. 静态成员类与非静态成员类之间唯一的区别是，静态成员类的声明中包含修饰符static。非静态成员类中的关联关系需要消耗非静态成员类实例的空间，并且增加了构造的时间开销。非静态成员类的一种常见用法是定义一个Adapter。
3. 匿名类的一种常见用法是动态地创建函数对象。
4. 局部类是使用最少的，在任何可以声明局部变量的地方，都可以声明局部类。并且局部类也遵守同样的作用域规则。

## 五. 泛型

### 1. 请不要在新代码中使用原生态类型

​	每个泛型都定义一个原生态类型，即不带任何实际类型参数的泛型名称。例如，List<E>相对应的原生态类型是List。

​	若不提供类型参数，使用集合类型和其他泛型仍然是合法的，但是不应该这么做。如果使用原生态类型，就失掉了泛型在安全性和表述性方面的所有优势，Java之所以仍然提供原生态类型使用，是因为需要兼容引入泛型直线的代码。

​	原生态类型List和参数化类型List<Object>之间的区别：前者逃避了泛型检查，后者则明确告诉编译器，它能够持有任意类型的对象。你可以讲List<String>传递给类型List的参数，但是不能将它传给类型List<Object>的参数。泛型有子类型化的规则，List<String>是原生类型List的一个子类型，而不是参数化类型List<Object>的子类型。

​	对于不确定或者不在乎集合中元素类型的情况，Java提供了一种安全的替代方法，称作无限制的通配符类型，例如List<?>。

​	无限制通配符类型Set<?>和原生态类型Set之间的区别：通配符类型是安全的，原生态类型则不安全。你可以将任何元素放入原生态类型的集合中，但不能将任何元素(除了null之外)放到Collection<?>中。

​	例外：在类文字中必须使用原生态类型。换句话说，List.class、String[].class合法，但是List<String>.class不合法。是因为泛型信息可以在运行时被擦除。

### 2. 消除非受检警告

在使用泛型编程时，有许多非受检警告很容易消除，例如

~~~java
//错误写法
Set<Lark> sets = new HashSet();
//改造后
Set<Lark> sets = new HashSet<Lark>();
~~~

但在使用泛型时，有些警告比较难以消除，但要坚持住，要尽可能地消除每个非受检警告。

SuppressWarning注解可以用在任何粒度的级别中，应该始终在尽可能小的范围内使用SuppressWarning注解。永远不要在类上使用该注解。

### 3. 列表优先于数组

​	数组是协变的，即如果Sub是Super的子类型，那么数组类型Sub[]就是Super[]的子类型

​	泛型则是不可变的，List<Type1>既不是List<Type2>的子类型，也不是List<Type2>的超类型

​	数组与泛型最大的区别就是，数组是具体化的，数组会在运行时才知道并检查它们的元素类型约束；相比之下，泛型则是通过擦除来实现的，因此泛型只在编译时强化它们的类型信息，并在运行时丢弃元素类型信息。所以创建一个泛型数组是不合法的。

​	一般来说，数组和泛型不能很好地混合使用，如果你发现自己将他们混合起来使用，优先使用列表代替数组。

### 4. 优先考虑泛型

在编写泛型类或者泛型方法时，有可能会遇到这种情况

~~~java
class Stack<E>{
    private E[] elements;
    ……
	elements = new E[16];
    ……
}
~~~

解决这个问题有两种解决办法：

1. 创建一个Object数组，将其转换成泛型数组类型。

~~~java
elements = (E[]) new Object[16];
~~~

2. 将elements域类型从E[]改为Object[]。

~~~java
private Object[] elements;
E result = (E)elements[--size]
~~~

​	使用泛型比使用需要在客户端中进行转换的类型来得更加安全，也更加容易。在设计新类型时，要确保它们不需要这种转换就可以使用，这通常意味着要把类做成泛型的。

### 5. 利用有限制通配符来提升API的灵活性



### 6. 优先考虑类型安全的异构容器





