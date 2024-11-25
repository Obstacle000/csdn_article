> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/weixin_46594796/article/details/118082268)

#### 文章目录

*   *   [1、前言](#1_2)
    *   [2、一个需求引出反射](#2_12)
    *   [3、反射机制](#3_73)
    *   *   [3.1 Java Reflection](#31_Java_Reflection_74)
        *   [3.2 反射机制原理图](#32__78)
        *   [3.3 反射机制可以完成的功能](#33__81)
        *   [3.4 反射相关的主要类](#34__88)
        *   [3.5 反射优点和缺点](#35__148)
        *   [3.6 反射调用优化——关闭访问检查](#36__152)
    *   [4、Class 类](#4Class_214)
    *   *   [4.1 基本介绍](#41__215)
        *   [4.2 Class 类的常用方法](#42_Class_225)
    *   [5、获取 Class 类对象](#5Class_282)
    *   [6、哪些类型有 Class 对象](#6Class_331)
    *   [7、类加载](#7_371)
    *   *   [7.1 基本说明](#71__372)
        *   [7.2 类加载时机](#72__377)
        *   [7.3 类加载过程图](#73__383)
        *   [7.4 类加载三个阶段完成任务](#74__386)
        *   *   [7.4.1 加载阶段](#741__389)
            *   [7.4.2 连接阶段——验证](#742__392)
            *   [7.4.3 连接阶段——准备](#743__395)
            *   [7.4.4 连接阶段——解析](#744__418)
            *   [7.4.5 Initialization（初始化）](#745_Initialization_421)
    *   [8、通过反射获取类的结构信息](#8_471)
    *   *   [8.1 java.lang.Class 类](#81_javalangClass_472)
        *   [8.2 java.lang.reflect.Field 类](#82_javalangreflectField_475)
        *   [8.3 java.lang.reflect.Method 类](#83_javalangreflectMethod_478)
        *   [8.4 java.lang.reflect.Constructor 类](#84_javalangreflectConstructor_481)
    *   [9、通过反射创建对象](#9_653)
    *   [10、通过反射访问类中的成员](#10_715)
    *   *   [10.1 访问属性](#101__716)
        *   [10.2 访问方法](#102__761)
    *   [11、练习案例](#11_826)
    *   *   [练习一](#_827)
        *   [练习二](#_869)

### 1、前言

该技术博客是我根据韩顺平老师的[反射机制](https://so.csdn.net/so/search?q=%E5%8F%8D%E5%B0%84%E6%9C%BA%E5%88%B6&spm=1001.2101.3001.7020)教程整理的学习笔记，希望能为大家带来帮助！链接如下：

[https://www.bilibili.com/video/BV1g84y1F7df?share_source=copy_web](https://www.bilibili.com/video/BV1g84y1F7df?share_source=copy_web)

> _**更多好文推荐**_ ：
> 
> *   [Java 常用类笔记](https://blog.csdn.net/weixin_46594796/article/details/113742367)
> *   [计算机底层原理——汇编语言](https://blog.csdn.net/weixin_46594796/article/details/108017547)
> *   [学习 Java 异常，吃透这篇足够](https://blog.csdn.net/weixin_46594796/article/details/113442988)

### 2、一个需求引出反射

需求如下：

> 根据配置文件 re.properties 中的指定信息，创建 Cat 对象并调用方法 hi  
> 配置文件中代码：`classfullpath=com.hspedu.Cat`，`method=hi`  
> 这样的需求在学习框架时非常多，通过外部文件配置，在不修改源码情况下控制程序  
> 符合设计模式的 ocp 原则（开闭原则：不修改源码，扩容功能）

代码如下：

```
package com.hspedu;

public class Cat {
    private String name = "招财猫";

    public void hi() {
        System.out.println("hi " + name);
    }

    public void cry() {
        System.out.println(name + "喵喵叫");
    }
}
==============================================================================
package com.hspedu.reflection.question;

import java.io.FileInputStream;
import java.lang.reflect.Method;
import java.util.Properties;

//反射问题引入
@SuppressWarnings({"all"}) //抑制警告信息
public class ReflectionQuestion {
    public static void main(String[] args) throws Exception {
        //根据配置文件中的信息，创建Cat对象并调用hi方法
        //传统方式：new Cat().hi();

        //1.使用Properties类读写配置文件
        Properties properties = new Properties();
        properties.load(new FileInputStream("src\\re.properties"));
        String classfullpath = properties.get("classfullpath").toString(); //com.hspedu.Cat
        String methodName = properties.get("method").toString(); //hi
        System.out.println("classfullpath=" + classfullpath);
        System.out.println("methodName=" + methodName);

        //2.使用传统方式创建对象行不通，需要使用 反射机制
        //2.1加载类，返回Class类型的对象
        Class cls = Class.forName(classfullpath);
        //2.2通过 cls对象 得到加载的类 com.hspedu.Cat 的对象
        Object o = cls.newInstance();
        //2.3通过 cls对象 得到加载的类 com.hspedu.Cat 的 methodName 方法对象
        //在反射机制中，可以把方法视为对象（万物皆对象）
        Method method1 = cls.getMethod(methodName);
        //2.4通过 method1 调用方法：即通过方法对象实现调用方法
        method1.invoke(o); //传统方法：对象.方法(); 反射机制：方法.invoke(对象)
    }
}

```

此时我们发现，我们只需要将 re.properties 中的 method=hi 改成 method=cry，就会调用 cry()，不需要修改源码，反射机制非常强大！

### 3、反射机制

#### 3.1 Java Reflection

1.  反射机制允许程序在执行期间借助于 Reflection API 取得任何类的内部信息（比如：成员变量，构造器，成员方法等），并能操作对象的属性及方法。反射在设计模式和框架的底层都会用到
2.  加载完类之后，在堆内存中就产生了一个 Class 类型的对象（一个类只有一个 Class 对象），这个对象包含了**类的完整结构信息**。通过这个对象得到类的结构。这个 Class 对象就像一面镜子，透过这个镜子看到类的结构，所以形象称之为：反射

#### 3.2 反射机制[原理图](https://so.csdn.net/so/search?q=%E5%8E%9F%E7%90%86%E5%9B%BE&spm=1001.2101.3001.7020)

![](https://i-blog.csdnimg.cn/blog_migrate/67d538d2bbbce78fa3461f4b5b8acff7.png)

#### 3.3 反射机制可以完成的功能

1.  在运行时，判断任意一个对象所属的类
2.  在运行时，创建任意一个类的对象
3.  在运行时，得到任意一个类所具有的成员变量和方法
4.  在运行时，调用任意一个对象的成员变量和方法
5.  生成动态代理

#### 3.4 反射相关的主要类

这些类在 java.lang.reflection 包下：

1.  java.lang.Class：代表一个类，Class 对象表示某个类加载后在堆内存中的对象
2.  java.lang.reflect.Method：代表类的方法，Method 对象表示某个类的方法
3.  java.lang.reflect.Field：代表类的成员变量，Field 对象表示某个类的成员变量
4.  java.lang.reflect.Constructor：代表类的构造方法，Constructor 对象表示构造器

代码展示如下：

```
public class Cat {
    private String name = "招财猫";
    public int age = 10;

    public Cat() {
    }

    public Cat(String name) {
        this.name = name;
    }

    public void hi() {
        System.out.println("hi " + name);
    }

    public void cry() {
        System.out.println(name + "喵喵叫");
    }
}
==========================================================================
import java.io.FileInputStream;
import java.lang.reflect.Constructor;
import java.lang.reflect.Field;
import java.util.Properties;

public class Reflection01 {
    public static void main(String[] args) throws Exception {
        Properties properties = new Properties();
        properties.load(new FileInputStream("src\\re.properties"));
        String classfullpath = properties.get("classfullpath").toString(); //com.hspedu.Cat
        String methodName = properties.get("method").toString(); //hi
        Class cls = Class.forName(classfullpath);
        Object o = cls.newInstance();

        //java.lang.reflect.Field：代表类的成员变量，Field对象表示某个类的成员变量
        //getField()不能得到私有的属性
        Field nameField = cls.getField("age");
        System.out.println(nameField.get(o));

        //java.lang.reflect.Constructor：代表类的构造方法，Constructor对象表示构造器
        //()中可以指定构造器的参数类型，如果不传参就是无参构造器
        Constructor constructor = cls.getConstructor();
        System.out.println(constructor);
        //参数传递String.class就是String类的Class对象
        Constructor constructor1 = cls.getConstructor(String.class);
        System.out.println(constructor1);
    }
}

```

#### 3.5 反射优点和缺点

*   优点：可以动态的创建和使用对象（就是框架底层核心），使用灵活，没有反射机制，框架技术就失去底层支撑。
*   缺点：使用反射基本是解释执行，对执行速度有影响

#### 3.6 反射调用优化——关闭访问检查

1.  Method、Field、Constructor 对象都有 setAccessible() 方法
2.  setAccessible() 作用是启动和禁用访问安全检查开关
3.  参数为 true 表示：反射的对象在使用时取消访问检查，提高反射的效率。  
    参数为 false 表示：反射的对象执行访问检查

![](https://i-blog.csdnimg.cn/blog_migrate/8841022f436d68776497681a8e6cf106.png)

代码展示如下：

```
import com.hspedu.Cat;
import java.lang.reflect.Method;

public class Reflection01 {
    public static void main(String[] args) throws Exception {
        m1(); //m1()耗时：4
        m2(); //m2()耗时：161
        m3(); //m3()耗时：109
    }

    //传统方式调用hi()，将hi()中的输出语句注释
    public static void m1() {
        Cat cat = new Cat();
        long start = System.currentTimeMillis();
        for (int i = 0; i < 90000000; i++) {
            cat.hi();
        }
        long end = System.currentTimeMillis();
        System.out.println("m1()耗时：" + (end - start));
    }

    //反射机制调用hi()，将hi()中的输出语句注释
    public static void m2() throws Exception {
        Class cls = Class.forName("com.hspedu.Cat");
        Object o = cls.newInstance();
        Method hi = cls.getMethod("hi");
        long start = System.currentTimeMillis();
        for (int i = 0; i < 90000000; i++) {
            hi.invoke(o);
        }
        long end = System.currentTimeMillis();
        System.out.println("m2()耗时：" + (end - start));
    }

    //反射调用优化：关闭访问检测
    public static void m3() throws Exception {
        Class cls = Class.forName("com.hspedu.Cat");
        Object o = cls.newInstance();
        Method hi = cls.getMethod("hi");
        //在反射调用方法时，取消访问检查
        hi.setAccessible(true);
        long start = System.currentTimeMillis();
        for (int i = 0; i < 90000000; i++) {
            hi.invoke(o);
        }
        long end = System.currentTimeMillis();
        System.out.println("m3()耗时：" + (end - start));
    }
}

```

### 4、Class 类

#### 4.1 基本介绍

![](https://i-blog.csdnimg.cn/blog_migrate/797fb2bb1cb7cfcbf1f76a41e612f5ad.png)

1.  Class 也是类，因此也继承 Object 类
2.  Class 类对象不是 new 出来的，而是系统创建的
3.  对于某个类的 Class 类对象，在内存中只有一份，因为类只加载一次
4.  每个类的实例都会记得自己是由哪个 Class 实例生成
5.  通过 Class 对象可以完整地得到一个类的完整结构，通过一系列 API
6.  Class 对象存放在堆内存中
7.  类的字节码二进制数据存放在方法区中，有的地方称为类的元数据（包括：方法，变量名，方法名，访问权限等）

#### 4.2 Class 类的常用方法

![](https://i-blog.csdnimg.cn/blog_migrate/77cd218f3c134a5abd85d6d05c3d878f.png)

代码展示如下：

```
public class Car {
    public String brand = "宝马";
    public int price = 500000;
    public String color = "白色";

    @Override
    public String toString() {
        return "Car{" +
                "brand='" + brand + '\'' +
                ", price=" + price +
                ", color='" + color + '\'' +
                '}';
    }
}
=======================================================================
import com.hspedu.Car;
import java.lang.reflect.Field;

//演示Class类的常用方法
public class Class02 {
    public static void main(String[] args) throws Exception {
        String classAllPath = "com.hspedu.Car";
        //1.获取Car类对应的Class对象
        //<?>表示不确定的Java类型
        Class<?> cls = Class.forName(classAllPath);
        //2.输出cls，显示cls对象是哪个类的Class对象
        System.out.println(cls); //com.hspedu.Car
        //输出cls运行类型 java.lang.Class
        System.out.println(cls.getClass());
        //3.获取包名，com.hspedu
        System.out.println(cls.getPackage().getName());
        //4.得到全类名，java.hspedu.Car
        System.out.println(cls.getName());
        //5.通过cls创建对象实例
        Car car = (Car) cls.newInstance();
        System.out.println(car);
        //6.通过反射获取属性 brand
        Field brand = cls.getField("brand");
        System.out.println(brand.get(car));
        //7.通过反射给属性赋值
        brand.set(car, "奔驰");
        System.out.println(brand.get(car));
        //8.获取所有的属性
        Field[] fields = cls.getFields();
        for (Field field : fields) {
            System.out.println(field.getName());
        }
    }
}

```

### 5、获取 Class 类对象

```
import com.hspedu.Car;

//得到Class对象的6种方式
public class GetClass_ {
    public static void main(String[] args) throws Exception {
        //1.Class.forName【编译阶段】，应用场景：通过读取配置文件获取
        String classAllPath = "com.hspedu.Car";
        Class cls1 = Class.forName(classAllPath);
        System.out.println(cls1);

        //2.类名.class【Class类阶段】，应用场景：用于参数传递
        Class cls2 = Car.class;
        System.out.println(cls2);

        //3.对象.getClass()【运行阶段】，应用场景：有对象实例
        Car car = new Car();
        Class cls3 = car.getClass();
        System.out.println(cls3);

        //4.通过类加载器（4种）【类加载阶段】来获取类的Class对象
        //4.1获取Car类的类加载器
        ClassLoader classLoader = car.getClass().getClassLoader();
        //4.2通过类加载器得到Class对象
        Class cls4 = classLoader.loadClass(classAllPath);
        System.out.println(cls4);

        //cls1,cls2,cls3,cls4都是同一个Class对象（一个类只有一个Class对象）
        System.out.println(cls1.hashCode());
        System.out.println(cls2.hashCode());
        System.out.println(cls3.hashCode());
        System.out.println(cls4.hashCode());

        //5.基本数据类型按照如下方式得到Class类对象
        Class<Integer> integerClass = int.class;
        Class<Character> characterClass = char.class;
        Class<Boolean> booleanClass = boolean.class;
        System.out.println(integerClass); //int

        //6.基本数据类型对应的包装类，可以通过.TYPE得到Class类对象
        Class<Integer> type1 = Integer.TYPE;
        Class<Character> type2 = Character.TYPE;
        System.out.println(type1); //int
    }
}

```

### 6、哪些类型有 Class 对象

1.  外部类，成员内部类，静态内部类，局部内部类，匿名内部类
2.  接口
3.  数组
4.  枚举
5.  注解
6.  基本数据类型
7.  void

代码展示如下：

```
import java.io.Serializable;

public class AllTypeClass {
    public static void main(String[] args) {
        Class<String> cls1 = String.class; //外部类
        Class<Serializable> cls2 = Serializable.class; //接口
        Class<Integer[]> cls3 = Integer[].class; //数组
        Class<float[][]> cls4 = float[][].class; //二维数组
        Class<Deprecated> cls5 = Deprecated.class; //注解
        Class<Thread.State> cls6 = Thread.State.class; //枚举
        Class<Long> cls7 = long.class; //基本数据类型
        Class<Void> cls8 = void.class; //void 数据类型
        Class<Class> cls9 = Class.class;

        System.out.println(cls1);
        System.out.println(cls2);
        System.out.println(cls3);
        System.out.println(cls4);
        System.out.println(cls5);
        System.out.println(cls6);
        System.out.println(cls7);
        System.out.println(cls8);
        System.out.println(cls9);
    }
}

```

### 7、[类加载](https://so.csdn.net/so/search?q=%E7%B1%BB%E5%8A%A0%E8%BD%BD&spm=1001.2101.3001.7020)

#### 7.1 基本说明

反射机制是 Java 实现**动态语言**的关键，也就是通过反射实现类的动态加载。

1.  静态加载：编译时加载相关的类，如果没有则报错，依赖性太强
2.  动态加载：运行时加载需要的类，如果运行时不用该类，即使不存在该类，也不报错，降低了依赖性

#### 7.2 类加载时机

1.  当创建对象时（new）——静态加载
2.  当子类被加载时，父类也加载——静态加载
3.  调用类中的静态成员时——静态加载
4.  通过反射——动态加载

#### 7.3 类加载过程图

![](https://i-blog.csdnimg.cn/blog_migrate/0bb4993479402aa148022da2f46227eb.png)

#### 7.4 类加载三个阶段完成任务

![](https://i-blog.csdnimg.cn/blog_migrate/b56a4842cc0e3be9464e1e945f6b0a47.png)

##### 7.4.1 加载阶段

![](https://i-blog.csdnimg.cn/blog_migrate/e52c3553683f054bbd83a87bcda646c6.png)

##### 7.4.2 连接阶段——验证

![](https://i-blog.csdnimg.cn/blog_migrate/edcd0ce4bb7fd810bcc595572d047d75.png)

##### 7.4.3 连接阶段——准备

![](https://i-blog.csdnimg.cn/blog_migrate/61047913888baa30e9f4a4d8834fd7ca.png)

```
//类加载的连接阶段——准备
public class ClassLoad02 {
    public static void main(String[] args) {
        //...
    }
}

class A {
    //属性（成员变量，字段）
    //分析类加载的连接阶段——准备，属性是如何处理：
    //1. n1是成员变量，不是静态变量，因此在准备阶段，不会分配内存
    //2. n2是静态变量，分配内存，n2是默认初始化0，而不是20
    //3. n3是static final常量，和静态变量不一样，因为一旦赋值就不变，n3 = 30
    public int n1 = 10;
    public static int n2 = 20;
    public static final int n3 = 30;
}

```

##### 7.4.4 连接阶段——解析

虚拟机将常量池内的符号引用替换为直接引用的过程。

##### 7.4.5 Initialization（初始化）

![](https://i-blog.csdnimg.cn/blog_migrate/78f0f56a6b830c5c2332e6558c28e003.png)  
代码展示如下：

```
//类加载初始化阶段
public class ClassLoad03 {
    public static void main(String[] args) throws Exception {
        //1.加载B类，并生成对应的Class类对象
        //2.连接 num = 0;
        //3.初始化阶段：依次自动收集类中的所有静态变量的赋值动作和静态代码块中的语句，并合并
        /*
            clinit(){
                System.out.println("B 静态代码块被执行");
                //num = 300;
                num = 100;
            }
            合并：num = 100;
         */

        //new B(); //类加载
        //System.out.println(B.num); //100，如果直接使用类的静态属性，也会导致类的加载

        //加载类的时候，是有同步机制控制
        /*
            protected Class<?> loadClass(String name, boolean resolve) throws ClassNotFoundException {
                //正因为有这个机制，才能保证某个类在内存中，只有一个 Class 对象
                synchronized (getClassLoadingLock(name)) {
                    //...
                }
            }
         */
        B b = new B();
    }
}

class B {
    static {
        System.out.println("B 静态代码块被执行");
        num = 300;
    }

    static int num = 100;

    public B() {
        System.out.println("B 构造器被执行");
    }
}

```

### 8、通过反射获取类的结构信息

#### 8.1 java.lang.Class 类

![](https://i-blog.csdnimg.cn/blog_migrate/902cbdedf3c8f523d40e4987ef364270.png)

#### 8.2 java.lang.reflect.Field 类

![](https://i-blog.csdnimg.cn/blog_migrate/160fe7ef62eade8c0076c5125eb0b5d8.png)

#### 8.3 java.lang.reflect.Method 类

![](https://i-blog.csdnimg.cn/blog_migrate/6ebf86fd7d2636c89a33419841619e5a.png)

#### 8.4 java.lang.reflect.Constructor 类

![](https://i-blog.csdnimg.cn/blog_migrate/4f3a04bc9a1da9186b0006f4283f2518.png)  
代码展示如下：

```
import org.junit.Test;
import java.lang.annotation.Annotation;
import java.lang.reflect.Constructor;
import java.lang.reflect.Field;
import java.lang.reflect.Method;

//通过反射获取类的结构信息
public class ReflectionUtils {
    @Test
    public void api_02() throws Exception {
        //得到Class对象
        Class<?> personCls = Class.forName("com.hspedu.reflection.Person");
        //getDeclaredFields:获取本类中所有属性
        //规定 说明: 默认修饰符 是0 ， public  是1 ，private 是 2 ，protected 是 4 , static 是 8 ，final 是 16
        Field[] declaredFields = personCls.getDeclaredFields();
        for (Field declaredField : declaredFields) {
            System.out.println("本类中所有属性=" + declaredField.getName()
                    + " 该属性的修饰符值=" + declaredField.getModifiers()
                    + " 该属性的类型=" + declaredField.getType());
        }

        //getDeclaredMethods:获取本类中所有方法
        Method[] declaredMethods = personCls.getDeclaredMethods();
        for (Method declaredMethod : declaredMethods) {
            System.out.println("本类中所有方法=" + declaredMethod.getName()
                    + " 该方法的访问修饰符值=" + declaredMethod.getModifiers()
                    + " 该方法返回类型" + declaredMethod.getReturnType());

            //输出当前这个方法的形参数组情况
            Class<?>[] parameterTypes = declaredMethod.getParameterTypes();
            for (Class<?> parameterType : parameterTypes) {
                System.out.println("该方法的形参类型=" + parameterType);
            }
        }

        //getDeclaredConstructors:获取本类中所有构造器
        Constructor<?>[] declaredConstructors = personCls.getDeclaredConstructors();
        for (Constructor<?> declaredConstructor : declaredConstructors) {
            System.out.println("====================");
            System.out.println("本类中所有构造器=" + declaredConstructor.getName());

            Class<?>[] parameterTypes = declaredConstructor.getParameterTypes();
            for (Class<?> parameterType : parameterTypes) {
                System.out.println("该构造器的形参类型=" + parameterType);
            }
        }
    }

    //第一组方法API
    @Test
    public void api_01() throws Exception {
        //得到Class对象
        Class<?> personCls = Class.forName("com.hspedu.reflection.Person");
        //getName:获取全类名
        System.out.println(personCls.getName());
        //getSimpleName:获取简单类名
        System.out.println(personCls.getSimpleName());
        //getFields:获取所有public修饰的属性，包含本类以及父类的
        Field[] fields = personCls.getFields();
        for (Field field : fields) {
            System.out.println("本类以及父类的属性=" + field.getName());
        }
        //getDeclaredFields:获取本类中所有属性
        Field[] declaredFields = personCls.getDeclaredFields();
        for (Field declaredField : declaredFields) {
            System.out.println("本类中所有属性=" + declaredField.getName());
        }
        //getMethods:获取所有public修饰的方法，包含本类以及父类的
        Method[] methods = personCls.getMethods();
        for (Method method : methods) {
            System.out.println("本类以及父类的方法=" + method.getName());
        }
        //getDeclaredMethods:获取本类中所有方法
        Method[] declaredMethods = personCls.getDeclaredMethods();
        for (Method declaredMethod : declaredMethods) {
            System.out.println("本类中所有方法=" + declaredMethod.getName());
        }
        //getConstructors: 获取所有public修饰的构造器，包含本类
        Constructor<?>[] constructors = personCls.getConstructors();
        for (Constructor<?> constructor : constructors) {
            System.out.println("本类的构造器=" + constructor.getName());
        }
        //getDeclaredConstructors:获取本类中所有构造器
        Constructor<?>[] declaredConstructors = personCls.getDeclaredConstructors();
        for (Constructor<?> declaredConstructor : declaredConstructors) {
            System.out.println("本类中所有构造器=" + declaredConstructor.getName());
        }
        //getPackage:以Package形式返回 包信息
        System.out.println(personCls.getPackage());
        //getSuperClass:以Class形式返回父类信息
        Class<?> superclass = personCls.getSuperclass();
        System.out.println("父类的class对象=" + superclass);
        //getInterfaces:以Class[]形式返回接口信息
        Class<?>[] interfaces = personCls.getInterfaces();
        for (Class<?> anInterface : interfaces) {
            System.out.println("接口信息=" + anInterface);
        }
        //getAnnotations:以Annotation[] 形式返回注解信息
        Annotation[] annotations = personCls.getAnnotations();
        for (Annotation annotation : annotations) {
            System.out.println("注解信息=" + annotation);
        }
    }
}

class A {
    public String hobby;

    public void hi() {

    }

    public A() {
        
    }

    public A(String name) {
        
    }
}

interface IA {
    
}

interface IB {

}

@Deprecated
class Person extends A implements IA, IB {
    public String name;
    protected static int age; // 4 + 8 = 12
    String job;
    private double sal;

    //构造器
    public Person() {
    }

    public Person(String name) {
    }

    //私有的
    private Person(String name, int age) {

    }

    //方法
    public void m1(String name, int age, double sal) {

    }

    protected String m2() {
        return null;
    }

    void m3() {

    }

    private void m4() {

    }
}

```

### 9、通过反射创建对象

![](https://i-blog.csdnimg.cn/blog_migrate/24be41014f080ea007188024bbe639a8.png)

测试 1：通过反射创建某类的对象，要求该类中必须有 public 的无参构造  
测试 2：通过调用某个特定构造器的方式，实现创建某类的对象

```
import java.lang.reflect.Constructor;

//通过反射机制创建实例
public class ReflectCreateInstance {
    public static void main(String[] args) throws Exception {
        //1. 先获取到User类的Class对象
        Class<?> userClass = Class.forName("com.hspedu.reflection.User");
        //2. 通过public的无参构造器创建实例
        Object o = userClass.newInstance();
        System.out.println(o);
        //3. 通过public的有参构造器创建实例
        /*
            constructor 对象就是
            public User(String name) {//public的有参构造器
                this.name = name;
            }
         */
        //3.1 先得到对应构造器
        Constructor<?> constructor = userClass.getConstructor(String.class);
        //3.2 创建实例，并传入实参
        Object hsp = constructor.newInstance("hsp");
        System.out.println("hsp=" + hsp);
        //4. 通过非public的有参构造器创建实例
        //4.1 得到private的构造器对象
        Constructor<?> constructor1 = userClass.getDeclaredConstructor(int.class, String.class);
        //4.2 创建实例
        //暴破【暴力破解】，使用反射可以访问private构造器/方法/属性, 反射面前，都是纸老虎
        constructor1.setAccessible(true);
        Object user2 = constructor1.newInstance(100, "张三丰");
        System.out.println("user2=" + user2);
    }
}

class User {
    private int age = 10;
    private String name = "韩顺平教育";

    public User() {
    }

    public User(String name) {
        this.name = name;
    }

    private User(int age, String name) {
        this.age = age;
        this.name = name;
    }

    public String toString() {
        return "User [age=" + age + ", ]";
    }
}

```

### 10、通过反射访问类中的成员

#### 10.1 访问属性

![](https://i-blog.csdnimg.cn/blog_migrate/9803e6ffd0db92ce0147f801c7a39be4.png)  
代码展示如下：

```
import java.lang.reflect.Field;

//反射操作属性
public class ReflectAccessProperty {
    public static void main(String[] args) throws Exception {
        //1. 得到Student类对应的 Class对象
        Class<?> stuClass = Class.forName("com.hspedu.reflection.Student");
        //2. 创建对象
        Object o = stuClass.newInstance();//o 的运行类型就是Student
        System.out.println(o.getClass());//Student
        //3. 使用反射得到age 属性对象
        Field age = stuClass.getField("age");
        age.set(o, 88);//通过反射来操作属性
        System.out.println(o);//
        System.out.println(age.get(o));//返回age属性的值
        //4. 使用反射操作name 属性
        Field name = stuClass.getDeclaredField("name");
        //对name 进行暴破, 可以操作private 属性
        name.setAccessible(true);
        //name.set(o, "老韩");
        name.set(null, "老韩~");//因为name是static属性，因此 o 也可以写出null
        System.out.println(o);
        System.out.println(name.get(o)); //获取属性值
        System.out.println(name.get(null));//获取属性值, 要求name是static
    }
}

class Student {
    public int age;
    private static String name;

    public Student() {
    }

    public String toString() {
        return "Student [age=" + age + ", ]";
    }
}

```

#### 10.2 访问方法

![](https://i-blog.csdnimg.cn/blog_migrate/56b8736fd9034b105b7a4d29ca27f30d.png)  
代码展示如下：

```
import java.lang.reflect.Method;

//通过反射调用方法
public class ReflectAccessMethod {
    public static void main(String[] args) throws Exception {
        //1. 得到Boss类对应的Class对象
        Class<?> bossCls = Class.forName("com.hspedu.reflection.Boss");
        //2. 创建对象
        Object o = bossCls.newInstance();
        //3. 调用public的hi方法
        //Method hi = bossCls.getMethod("hi", String.class);//OK
        //3.1 得到hi方法对象
        Method hi = bossCls.getDeclaredMethod("hi", String.class);//OK
        //3.2 调用
        hi.invoke(o, "我是徐志斌~");
        //4. 调用private static 方法
        //4.1 得到 say 方法对象
        Method say = bossCls.getDeclaredMethod("say", int.class, String.class, char.class);
        //4.2 因为say方法是private, 所以需要暴破，原理和前面讲的构造器和属性一样
        say.setAccessible(true);
        System.out.println(say.invoke(o, 100, "张三", '男'));
        //4.3 因为say方法是static的，还可以这样调用 ，可以传入null
        System.out.println(say.invoke(null, 200, "李四", '女'));

        //5. 在反射中，如果方法有返回值，统一返回Object , 但是他运行类型和方法定义的返回类型一致
        Object reVal = say.invoke(null, 300, "王五", '男');
        System.out.println("reVal 的运行类型=" + reVal.getClass());//String

        //在演示一个返回的案例
        Method m1 = bossCls.getDeclaredMethod("m1");
        Object reVal2 = m1.invoke(o);
        System.out.println("reVal2的运行类型=" + reVal2.getClass());//Monster
    }
}

class Monster {

}

class Boss {
    public int age;
    private static String name;

    public Boss() {
    }

    public Monster m1() {
        return new Monster();
    }

    private static String say(int n, String s, char c) {
        return n + " " + s + " " + c;
    }

    public void hi(String s) {
        System.out.println("hi " + s);
    }
}

```

### 11、练习案例

#### 练习一

![](https://i-blog.csdnimg.cn/blog_migrate/7079245408c1d1f3f728e51af0b64df3.png)  
代码展示如下：

```
import java.lang.reflect.Field;
import java.lang.reflect.Method;

public class HomeWork01 {
    public static void main(String[] args) throws Exception {
        /**
         * 定义PrivateTest类，有私有name属性，并且属性值为hellokitty
         * 提供getName的公有方法
         * 创建PrivateTest的类，利用Class类得到私有的name属性，修改私有的name属性值，并调用getName()的方法打印name属性值
         */
        //1. 得到 PrivateTest类对应的Class对象
        Class<PrivateTest> privateTestClass = PrivateTest.class;
        //2. 创建对象实例
        PrivateTest privateTestObj = privateTestClass.newInstance();
        //3. 得到name属性对象
        Field name = privateTestClass.getDeclaredField("name");//name属性是private
        //4. 暴破name
        name.setAccessible(true);
        name.set(privateTestObj, "天龙八部");
        //5. 得到getName方法对象
        Method getName = privateTestClass.getMethod("getName");
        //6. 因为getName() 是public，所有直接调用
        Object invoke = getName.invoke(privateTestObj);
        System.out.println("name属性值=" + invoke);//天龙八部
    }
}

class PrivateTest {
    private String name = "hellokitty";

    //默认无参构造器
    public String getName() {
        return name;
    }
}

```

#### 练习二

![](https://i-blog.csdnimg.cn/blog_migrate/571ea9a46b7cc6d22c02b09a9c11c5bb.png)  
代码展示如下：

```
import java.lang.reflect.Constructor;
import java.lang.reflect.Method;

public class HomeWork02 {
    public static void main(String[] args) throws Exception {
        /**
         * 利用Class类的forName方法得到File类的class 对象
         * 在控制台打印File类的所有构造器
         * 通过newInstance的方法创建File对象，并创建D:\mynew.txt文件
         */
        //1. Class类的forName方法得到File类的class 对象
        Class<?> fileCls = Class.forName("java.io.File");
        //2. 得到所有的构造器
        Constructor<?>[] declaredConstructors = fileCls.getDeclaredConstructors();
        for (Constructor<?> declaredConstructor : declaredConstructors) {
            System.out.println("File构造器=" + declaredConstructor);
        }
        //3. 指定的得到 public java.io.File(java.lang.String)
        Constructor<?> declaredConstructor = fileCls.getDeclaredConstructor(String.class);
        String fileAllPath = "d:\\mynew.txt";
        Object file = declaredConstructor.newInstance(fileAllPath);//创建File对象
        //4. 得到createNewFile 的方法对象
        Method createNewFile = fileCls.getMethod("createNewFile");
        createNewFile.invoke(file);//创建文件，调用的是 createNewFile
        //file的运行类型就是File
        System.out.println(file.getClass());
        System.out.println("创建文件成功" + fileAllPath);
    }
}

```