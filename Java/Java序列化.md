# Java序列化

## 1，serialVersionUID作用

serialVersionUID适用于Java的序列化机制。简单来说，Java的序列化机制是通过判断类的serialVersionUID来验证版本一致性的。在进行反序列化时，JVM会把传来的字节流中的serialVersionUID与本地相应实体类的serialVersionUID进行比较，如果相同就认为是一致的，可以进行反序列化，否则就会出现序列化版本不一致的异常，即是**InvalidCastException**。

**serialVersionUID有两种显示的生成方式**：        
一是默认的1L，比如：private static final long serialVersionUID = 1L;        
二是根据类名、接口名、成员方法及属性等来生成一个64位的哈希字段，比如：        
private static final  long   serialVersionUID = xxxxL;

当一个类实现了Serializable接口，如果没有显示的定义serialVersionUID，Eclipse会提供相应的提醒。面对这种情况，我们只需要在Eclipse中点击类中warning图标一下，Eclipse就会      自动给定两种生成的方式。如果不想定义，在Eclipse的设置中也可以把它关掉的，设置如下：
Window ==> Preferences ==> Java ==> Compiler ==> Error/Warnings ==> Potential programming problems
将Serializable class without serialVersionUID的warning改成ignore即可。

当实现java.io.Serializable接口的类没有显式地定义一个serialVersionUID变量时候，Java序列化机制会根据编译的Class自动生成一个serialVersionUID作序列化版本比较用，这种情况下，如果Class文件(类名，方法明等)没有发生变化(增加空格，换行，增加注释等等)，就算再编译多次，serialVersionUID也不会变化的。

如果我们不希望通过编译来强制划分软件版本，即实现序列化接口的实体能够兼容先前版本，就需要显式地定义一个名为serialVersionUID，类型为long的变量，不修改这个变量值的序列化实体都可以相互进行串行化和反串行化。

**假如对象新增字段，则反序列化不受影响，只是新加的会被忽略。简单理解就是个接口文档，版本变动，就是全部变动，新增和修改，只要我能兼容，就ok**.

清单 2 中的 main 方法，将对象序列化后，修改静态变量的数值，再将序列化对象读取出来，然后通过读取出来的对象获得静态变量的数值并打印出来。依照清单 2，这个 System.out.println(t.staticVar) 语句输出的是 10 还是 5 呢？

最后的输出是 10，对于无法理解的读者认为，打印的 staticVar 是从读取的对象里获得的，应该是保存时的状态才对。之所以打印 10 的原因在于序列化时，并不保存静态变量，这其实比较容易理解，序列化保存的是对象的状态，静态变量属于类的状态，因此 **序列化并不保存静态变量**。



情境：一个子类实现了 Serializable 接口，它的父类都没有实现 Serializable 接口，序列化该子类对象，然后反序列化后输出父类定义的某变量的数值，该变量数值与序列化时的数值不同。

解决：要想将父类对象也序列化，就需要让父类也实现Serializable 接口。如果父类不实现的话的，就 需要有默认的无参的构造函数。在父类没有实现 Serializable 接口时，虚拟机是不会序列化父对象的，而一个 Java 对象的构造必须先有父对象，才有子对象，反序列化也不例外。所以反序列化时，为了构造父对象，只能调用父类的无参构造函数作为默认的父对象。因此当我们取父对象的变量值时，它的值是调用父类无参构造函数后的值。如果你考虑到这种序列化的情况，在父类无参构造函数中对变量进行初始化，否则的话，父类变量值都是默认声明的值，如 int 型的默认是 0，string 型的默认是 null。



**Transient** 关键字的作用是控制变量的序列化，在变量声明前加上该关键字，可以阻止该变量被序列化到文件中，在被反序列化后，transient 变量的值被设为初始值，如 int 型的是 0，对象型的是 null。

我们熟悉使用 Transient 关键字可以使得字段不被序列化，那么还有别的方法吗？根据父类对象序列化的规则，我们可以将不需要被序列化的字段抽取出来放到父类中，子类实现 Serializable 接口，父类不实现，根据父类序列化规则，父类的字段数据将不被序列化，形成类图如图 2 所示。

https://www.cnblogs.com/duanxz/p/3511695.html

## Hessian序列化：

是这一种支持动态类型，跨语言，基于对象网路传输的协议。可以被C++,Python等反序列化。比原生的二进制大小减少50%



## 3，JSON序列化：

javascript Object Notation,轻量级的数据交互格式。

## 4，transient

```java
//标记该属性不能被序列化
private transient int age;
```

## 5，ObjectOutputStram

ObjectOutputStream

ObjectInputStream

我们一般使用ObjectOutputStream的`writeObject`方法把一个对象进行持久化。再使用ObjectInputStream的`readObject`从持久化存储中把对象读取出来。

## 6，Externalizble

Externalizable继承了Serializable，该接口中定义了两个抽象方法：`writeExternal()`与`readExternal()`。当使用Externalizable接口来进行序列化与反序列化的时候需要开发人员重写`writeExternal()`与`readExternal()`方法。由于上面的代码中，并没有在这两个方法中定义序列化实现细节，所以输出的内容为空。



## 7，总结

1、在Java中，只要一个类实现了`java.io.Serializable`接口，那么它就可以被序列化。

2、通过`ObjectOutputStream`和`ObjectInputStream`对对象进行序列化及反序列化

3、虚拟机是否允许反序列化，不仅取决于类路径和功能代码是否一致，一个非常重要的一点是两个类的序列化 ID 是否一致（就是 `private static final long serialVersionUID`）

4、序列化并不保存静态变量。

5、要想将父类对象也序列化，就需要让父类也实现`Serializable` 接口。

6、Transient 关键字的作用是控制变量的序列化，在变量声明前加上该关键字，可以阻止该变量被序列化到文件中，在被反序列化后，transient 变量的值被设为初始值，如 int 型的是 0，对象型的是 null。

7、服务器端给客户端发送序列化对象数据，对象中有一些数据是敏感的，比如密码字符串等，希望对该密码字段在序列化时，进行加密，而客户端如果拥有解密的密钥，只有在客户端进行反序列化时，才可以对密码进行读取，这样可以一定程度保证序列化对象的数据安全。

8、ArrayList实现了Serializable接口，但是

```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{
    private static final long serialVersionUID = 8683452581122892189L;
  //元素不能被序列化
    transient Object[] elementData; // non-private to simplify nested class access
    private int size;
}
```

实际情况这里虽然限制了，但是它复写了writeObject 和 readObject 方法。所以可以序列化。

ArrayList实际上是动态数组，每次在放满以后自动增长设定的长度值，如果数组自动增长长度设为100，而实际只放了一个元素，那就会序列化99个null元素。为了保证在序列化的时候不会将这么多null同时进行序列化，ArrayList把元素数组设置为transient。

