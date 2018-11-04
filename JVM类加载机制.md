# JVM类加载机制

JVM类加载机制分为五个部分：加载，验证，准备，解析，初始化，下面我们就分别来看一下这五个过程。

[![4003106-dd3ff46b4f9f3574](http://incdn1.b0.upaiyun.com/2017/06/2fb054008ca2898e0a17f7d79ce525a1.png)](http://www.importnew.com/25295.html/4003106-dd3ff46b4f9f3574)

## 加载

加载是类加载过程中的一个阶段，这个阶段会在内存中生成一个代表这个类的java.lang.Class对象，作为方法区这个类的各种数据的入口。注意这里不一定非得要从一个Class文件获取，这里既可以从ZIP包中读取（比如从jar包和war包中读取），也可以在运行时计算生成（动态代理），也可以由其它文件生成（比如将JSP文件转换成对应的Class类）。

## 验证

这一阶段的主要目的是为了**确保Class文件的字节流中包含的信息是否符合当前虚拟机的要求，并且不会危害虚拟机自身的安全**。

## 准备

准备阶段是**正式为类变量分配内存并设置类变量的初始值阶段**，即在**方法区中分配这些变量所使用的内存空间**。注意这里所说的初始值概念，比如一个类变量定义为：

```java
`public` `static` `int` `v = ``8080``;`
```

实际上变量v在准备阶段过后的初始值为0而不是8080，将v赋值为8080的putstatic指令是程序被编译后，存放于类构造器<client>方法之中，这里我们后面会解释。
但是注意如果声明为：

```java
`public` `static` `final` `int` `v = ``8080``;`
```

在编译阶段会为v生成ConstantValue属性，在准备阶段虚拟机会根据ConstantValue属性将v赋值为8080。

## 解析

解析阶段是指虚拟机将常量池中的符号引用替换为直接引用的过程。符号引用就是class文件中的：

- CONSTANT_Class_info
- CONSTANT_Field_info
- CONSTANT_Method_info

等类型的常量。

下面我们解释一下符号引用和直接引用的概念：

- 符号引用与虚拟机实现的布局无关，引用的目标并不一定要已经加载到内存中。各种虚拟机实现的内存布局可以各不相同，但是它们能接受的符号引用必须是一致的，因为符号引用的字面量形式明确定义在Java虚拟机规范的Class文件格式中。
- 直接引用可以是指向目标的指针，相对偏移量或是一个能间接定位到目标的句柄。如果有了直接引用，那引用的目标必定已经在内存中存在。

## 初始化

初始化阶段是类加载最后一个阶段，前面的类加载阶段之后，除了在加载阶段可以自定义类加载器以外，其它操作都由JVM主导。到了初始阶段，才开始真正执行类中定义的Java程序代码。

初始化阶段是执行类构造器<client>方法的过程。<client>方法是由编译器自动收集类中的类变量的赋值操作和静态语句块中的语句合并而成的。虚拟机会保证<client>方法执行之前，父类的<client>方法已经执行完毕。p.s: 如果一个类中没有对静态变量赋值也没有静态语句块，那么编译器可以不为这个类生成<client>()方法。

注意以下几种情况不会执行类初始化：

- 通过子类引用父类的静态字段，只会触发父类的初始化，而不会触发子类的初始化。
- 定义对象数组，不会触发该类的初始化。
- 常量在编译期间会存入调用类的常量池中，本质上并没有直接引用定义常量的类，不会触发定义常量所在的类。
- 通过类名获取Class对象，不会触发类的初始化。
- 通过Class.forName加载指定类时，如果指定参数initialize为false时，也不会触发类初始化，其实这个参数是告诉虚拟机，是否要对类进行初始化。
- 通过ClassLoader默认的loadClass方法，也不会触发初始化动作。

## 类加载器

虚拟机设计团队把加载动作放到JVM外部实现，以便让应用程序决定如何获取所需的类，JVM提供了3种类加载器：

- 启动类加载器(Bootstrap ClassLoader)：负责加载 JAVA_HOME\lib 目录中的，或通过-Xbootclasspath参数指定路径中的，且被虚拟机认可（按文件名识别，如rt.jar）的类。
- 扩展类加载器(Extension ClassLoader)：负责加载 JAVA_HOME\lib\ext 目录中的，或通过java.ext.dirs系统变量指定路径中的类库。
- 应用程序类加载器(Application ClassLoader)：负责加载用户路径（classpath）上的类库。

JVM通过双亲委派模型进行类的加载，当然我们也可以通过继承java.lang.ClassLoader实现自定义的类加载器。

[![JVM类加载机制-类加载器](./image/image-JVM类加载机制-类加载器.png)](JVM类加载机制-类加载器)

当一个类加载器收到类加载任务，会先交给其父类加载器去完成，因此最终加载任务都会传递到顶层的启动类加载器，只有当父类加载器无法完成加载任务时，才会尝试执行加载任务。

采用双亲委派的一个好处是比如加载位于rt.jar包中的类java.lang.Object，不管是哪个加载器加载这个类，最终都是委托给顶层的启动类加载器进行加载，这样就保证了使用不同的类加载器最终得到的都是同样一个Object对象。

## 类加载器

虚拟机设计团队把加载动作放到JVM外部实现，以便让应用程序决定如何获取所需的类，JVM提供了3种类加载器：

- 启动类加载器(Bootstrap ClassLoader)：负责加载 JAVA_HOME\lib 目录中的，或通过-Xbootclasspath参数指定路径中的，且被虚拟机认可（按文件名识别，如rt.jar）的类。
- 扩展类加载器(Extension ClassLoader)：负责加载 JAVA_HOME\lib\ext 目录中的，或通过java.ext.dirs系统变量指定路径中的类库。
- 应用程序类加载器(Application ClassLoader)：负责加载用户路径（classpath）上的类库。

JVM通过双亲委派模型进行类的加载，当然我们也可以通过继承java.lang.ClassLoader实现自定义的类加载器。

[![4003106-7e769584defe288c](G:\mine\笔记\d330251551f6de988239494ce2773095.png)](http://www.importnew.com/25295.html/4003106-7e769584defe288c)

当一个类加载器收到类加载任务，会先交给其父类加载器去完成，因此最终加载任务都会传递到顶层的启动类加载器，只有当父类加载器无法完成加载任务时，才会尝试执行加载任务。

采用双亲委派的一个好处是比如加载位于rt.jar包中的类java.lang.Object，不管是哪个加载器加载这个类，最终都是委托给顶层的启动类加载器进行加载，这样就保证了使用不同的类加载器最终得到的都是同样一个Object对象。

在有些情境中可能会出现要我们自己来实现一个类加载器的需求，由于这里涉及的内容比较广泛，我想以后单独写一篇文章来讲述，不过这里我们还是稍微来看一下。我们直接看一下jdk中的ClassLoader的源码实现：

```java
`protected` `synchronized` `Class<?> loadClass(String name, ``boolean` `resolve)``        ``throws` `ClassNotFoundException {``    ``// First, check if the class has already been loaded``    ``Class c = findLoadedClass(name);``    ``if` `(c == ``null``) {``        ``try` `{``            ``if` `(parent != ``null``) {``                ``c = parent.loadClass(name, ``false``);``            ``} ``else` `{``                ``c = findBootstrapClass0(name);``            ``}``        ``} ``catch` `(ClassNotFoundException e) {``            ``// If still not found, then invoke findClass in order``            ``// to find the class.``            ``c = findClass(name);``        ``}``    ``}``    ``if` `(resolve) {``        ``resolveClass(c);``    ``}``    ``return` `c;``}`
```

- 首先通过Class c = findLoadedClass(name);判断一个类是否已经被加载过。
- 如果没有被加载过执行if (c == null)中的程序，遵循双亲委派的模型，首先会通过递归从父加载器开始找，直到父类加载器是Bootstrap ClassLoader为止。
- 最后根据resolve的值，判断这个class是否需要解析。

而上面的findClass()的实现如下，直接抛出一个异常，并且方法是protected，很明显这是留给我们开发者自己去实现的，这里我们以后我们单独写一篇文章来讲一下如何重写findClass方法来实现我们自己的类加载器。

```java
`protected` `Class<?> findClass(String name) ``throws` `ClassNotFoundException {``    ``throw` `new` `ClassNotFoundException(name);``}`
```

## 双亲委派模型

### 1.1 什么是双亲委派模型？

首先，先要知道什么是类加载器。简单说，类加载器就是根据指定全限定名称将class文件加载到JVM内存，转为Class对象。如果站在JVM的角度来看，只存在两种类加载器:

#### 启动类加载器（Bootstrap ClassLoader）：

由C++语言实现（针对HotSpot）,负责将存放在\lib目录或-Xbootclasspath参数指定的路径中的类库加载到内存中。

#### 其他类加载器：

由Java语言实现，继承自抽象类ClassLoader。如：

##### 扩展类加载器（Extension ClassLoader）：

负责加载\lib\ext目录或java.ext.dirs系统变量指定的路径中的所有类库。

##### 应用程序类加载器（Application ClassLoader）

负责加载用户类路径（classpath）上的指定类库，我们可以直接使用这个类加载器。一般情况，如果我们没有自定义类加载器默认就是用这个加载器。

**双亲委派模型工作**过程是：如果一个类加载器收到类加载的请求，它首先不会自己去尝试加载这个类，而是把这个请求委派给父类加载器完成。每个类加载器都是如此，只有当父加载器在自己的搜索范围内找不到指定的类时（即ClassNotFoundException），子加载器才会尝试自己去加载。



### 1.2 为什么需要双亲委派模型？

假设没有双亲委派模型，试想一个场景：

- 黑客自定义一个**java.lang.String**类，该**String**类具有系统的**String类**一样的功能，只是在某个函数稍作修改。比如equals函数，这个函数经常使用，如果在这这个函数中，黑客加入一些“病毒代码”。并且通过自定义类加载器加入到**JVM**中。此时，如果没有双亲委派模型，那么JVM就可能误以为黑客**自定义的java.lang.String类是系统的String类**，导致“病毒代码”被执行。

而有了双亲委派模型，黑客自定义的java.lang.String类永远都不会被加载进内存。因为首先是最顶端的类加载器加载系统的java.lang.String类，最终自定义的类加载器无法加载java.lang.String类。

或许你会想，我在自定义的类加载器里面强制加载自定义的java.lang.String类，不去通过调用父加载器不就好了吗?确实，这样是可行。但是，**在JVM中，判断一个对象是否是某个类型时，如果该对象的实际类型与待比较的类型的类加载器不同，那么会返回false。**

举个简单例子：

ClassLoader1、ClassLoader2都加载java.lang.String类，对应Class1、Class2对象。那么Class1对象不属于ClassLoad2对象加载的java.lang.String类型。

### 1.3 如何实现双亲委派模型？

双亲委派模型的原理很简单，实现也简单。每次通过先委托父类加载器加载，当父类加载器无法加载时，再自己加载。其实ClassLoader类默认的loadClass方法已经帮我们写好了，我们无需去写。

#### 自定义类加载器中的几个重要函数

##### loadClass

loadClass默认实现如下：

```java
public Class loadClass(String name) throws ClassNotFoundException {
        return loadClass(name, false);
}
```


再看看loadClass(String name, boolean resolve)函数：

```java
protected Class loadClass(String name, boolean resolve)
    throws ClassNotFoundException
{
    synchronized (getClassLoadingLock(name)) {
        // First, check if the class has already been loaded
        Class c = findLoadedClass(name);
        if (c == null) {
            long t0 = System.nanoTime();
            try {
                if (parent != null) {
                    c = parent.loadClass(name, false);
                } else {
                    c = findBootstrapClassOrNull(name);
                }
            } catch (ClassNotFoundException e) {
                // ClassNotFoundException thrown if class not found
                // from the non-null parent class loader
            }
            if (c == null) {
            // If still not found, then invoke findClass in order
            // to find the class.
            long t1 = System.nanoTime();
            c = findClass(name);

            // this is the defining class loader; record the stats
            sun.misc.PerfCounter.getParentDelegationTime().addTime(t1 - t0);
            sun.misc.PerfCounter.getFindClassTime().addElapsedTimeFrom(t1);
            sun.misc.PerfCounter.getFindClasses().increment();
        }
    }
    if (resolve) {
        resolveClass(c);
    }
    return c;
}
```


从上面代码可以明显看出，loadClass(String, boolean)函数即实现了双亲委派模型！

整个大致过程如下：

- 首先，检查一下指定名称的类是否已经加载过，如果加载过了，就不需要再加载，直接返回。
- 如果此类没有加载过，那么，再判断一下是否有父加载器；如果有父加载器，则由父加载器加载（即调用parent.loadClass(name, false);）.或者是调用bootstrap类加载器来加载。
- 如果父加载器及bootstrap类加载器都没有找到指定的类，那么调用当前类加载器的findClass方法来完成类加载。换话句话说，如果自定义类加载器，就必须重写findClass方法！

##### find Class

findClass的默认实现如下：

```java
protected Class findClass(String name) throws ClassNotFoundException {
        throw new ClassNotFoundException(name);
}
```

可以看出，抽象类ClassLoader的findClass函数默认是抛出异常的。而前面我们知道，loadClass在父加载器无法加载类的时候，就会调用我们自定义的类加载器中的findeClass函数，因此我们必须要在loadClass这个函数里面实现将一个指定类名称转换为Class对象.

如果是是读取一个指定的名称的类为字节数组的话，这很好办。但是如何将字节数组转为Class对象呢？很简单，Java提供了defineClass方法，通过这个方法，就可以把一个字节数组转为Class对象啦~

##### defineClass

defineClass主要的功能是：

将一个字节数组转为Class对象，这个字节数组是class文件读取后最终的字节数组。如，假设class文件是加密过的，则需要解密后作为形参传入defineClass函数。

defineClass默认实现如下：

```java
protected final Class defineClass(String name, byte[] b, int off, int len)
        throws ClassFormatError  {
        return defineClass(name, b, off, len, null);
}
```



#### 函数调用过程

上一节所提的函数调用过程如下：

2.2 函数调用过程
上一节所提的函数调用过程如下：

##### 简单示例

首先，我们定义一个待加载的普通Java类:Test.java。放在com.huachao.cl包下:

```java
package com.huachao.cl;
public class Test {
    public void hello() {
        System.out.println("恩，是的，我是由 " + getClass().getClassLoader().getClass()
" 加载进来的");
}
```

注意：

如果你是直接在当前项目里面创建，待Test.java编译后，请把Test.class文件拷贝走，再将Test.java删除。因为如果Test.class存放在当前项目中，根据双亲委派模型可知，会通过sun.misc.Launcher$AppClassLoader 类加载器加载。为了让我们自定义的类加载器加载，我们把Test.class文件放入到其他目录。

在本例中，我们Test.class文件存放的目录如下：

接下来就是自定义我们的类加载器：

```java
import java.io.FileInputStream;
import java.lang.reflect.Method;

public class Main {
     static class MyClassLoader extends ClassLoader {
        private String classPath;
		 public MyClassLoader(String classPath) {
        this.classPath = classPath;
    }

    private byte[] loadByte(String name) throws Exception {
        name = name.replaceAll("\\.", "/");
        FileInputStream fis = new FileInputStream(classPath + "/" + name
                + ".class");
        int len = fis.available();
        byte[] data = new byte[len];
        fis.read(data);
        fis.close();
        return data;

    }

    protected Class findClass(String name) throws ClassNotFoundException {
        try {
            byte[] data = loadByte(name);
            return defineClass(name, data, 0, data.length);
        } catch (Exception e) {
            e.printStackTrace();
            throw new ClassNotFoundException();
        }
    }

};

public static void main(String args[]) throws Exception {
    MyClassLoader classLoader = new MyClassLoader("D:/test");
    Class clazz = classLoader.loadClass("com.huachao.cl.Test");
    Object obj = clazz.newInstance();
    Method helloMethod = clazz.getDeclaredMethod("hello", null);
    helloMethod.invoke(obj, null);
}

```

最后运行结果如下：

- 恩，是的，我是由 class Main$MyClassLoader 加载进来的