## 1. 类加载的过程
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1753727835886-355206b7-87d5-49f3-9ac0-ec7fbc776f22.png)

加载：将字节码文件通过IO流读取到JVM的方法区，并同时在堆中生成Class对像。

验证：校验字节码文件的正确性。

准备：为类的静态变量分配内存，并初始化为默认值；对于final static修饰的变量，在编译时就已经分配好内存了。

解析：将类中的符号引用转换为直接引用。

初始化：对类的静态变量初始化为指定的值，执行静态代码。

## 2. 类加载器的分类
**主要分为两类:**

1. JVM内置的类加载器,有Bootstrap加载器、ExtClassLoader加载器和AppClassLoader加载器 三种,分别负责加载不同目录下的.class文件

2. 用户自定义的类加载器,负责的加载目录自己决定。

### 2.1 引导类加载器 Bootstrap
引导类加载器属于JVM的一部分，由C++代码实现。

引导类加载器负责加载`<JAVA_HOME\>\jre\lib`路径下的核心类库，由于安全考虑只加载 包名 java、javax、sun开头的类。

```java
package classloader.bootstrap;

import sun.misc.Launcher;
import java.net.URL;

public class Demo01 {
    public static void main(String[] args) {
        // 1. 获取Object类的类加载器
        // Object类位于java.lang包中，是Java核心API的一部分
        // 它是由引导类加载器加载的
        ClassLoader classLoader = Object.class.getClassLoader();
        
        // 2. 打印Object类的类加载器
        // 由于引导类加载器是由C++实现的，在Java中没有对应的ClassLoader对象
        // 所以这里会输出null，表示是引导类加载器
        System.out.println(classLoader); // 输出：null
        
        // 3. 获取引导类加载器的加载路径
        // Launcher是JDK内部类，用于启动Java应用程序
        // getBootstrapClassPath()方法返回引导类加载器的类路径
        URL[] urLs = Launcher.getBootstrapClassPath().getURLs();
        
        // 4. 遍历并打印所有引导类路径
        for (URL urL : urLs) {
            // 打印引导类加载器加载的jar包或目录路径
            // 通常包括：JDK安装目录下的jre/lib/rt.jar等核心库
            System.out.println(urL);
        }
    }
}
```

### 2.2 扩展类加载器 ExtClassLoader
全类名:`sum.misc.Launch$ExtClassLoader`，Java语言实现。

扩展类加载器的父加载器是`Bootstrap启动类加载器` (注:不是继承关系)

扩展类加载器负责加载`<JAVA_HOME>\jre\lib\ext`目录下的类库。

**加载的jar包**  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1753727835915-9bbbd889-100e-4f04-8ed3-8616c31f86c5.png)  
**获取扩展类加载器**  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1753727835949-2f365063-27f0-4b4a-9f2e-cd0cd60812ab.png)  
**注:** JDK9是`jdk.internal.loader.ClassLoaders$PlatformClassLoader`类

### 2.3 系统类加载器 AppClassLoader
全类名: `sun.misc.Launcher$AppClassLoader `

系统类加载器的父加载器是`ExtClassLoader扩展类加载器`（注: 不是继承关系）。

系统类加载器负责加载 `classpath环境变量`所指定的类库，是用户自定义类的默认类加载器。

**获取系统类加载器**  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1753727835956-9018bd76-fff5-4175-b2cb-5263f6e39326.png)  
**注:** JDK9是`jdk.internal.loader.ClassLoaders$AppClassLoader`类

### 2.4 三者之间的关系
> AppClassLoader的父加载器是ExtClassLoader
>
> ExtClassLoader的父加载器是Bootstrap
>
> Bootstrap是根加载器
>
> 
>
> 三者之间是没有继承关系的。
>

:::color3
AppClassLoader和ExtClassLoader都实现了抽象类ClassLoader。

抽象类ClassLoader有一个字段parent, AppClassLoader和ExtClassLoader通过设置该字段引用,指定父加载器。（是组合关系）

AppClassLoader 的parent指向 ExtClassLoader

ExtClassLoader 的parent指向 null,(null的原因是因为Bootstrap是C++实现的,通过代码中逻辑判断来转向Bootstrap)



private final ClassLoader parent;

:::

### 2.5 自定义类加载器
**自定义类加载器是为了加载在jvm三个加载器负责的目录范围之外的类**

```java
package com;

import java.io.*;

public class MyClassLoader extends ClassLoader {
    private String classPath;  // 类文件的根目录路径

    // 构造方法1：使用默认的父类加载器
    public MyClassLoader(String classPath) {
        this.classPath = classPath;
    }

    // 构造方法2：指定父类加载器
    public MyClassLoader(ClassLoader parent, String classPath) {
        super(parent);  // 调用父类构造方法，设置父类加载器
        this.classPath = classPath;
    }

    @Override
    protected Class<?> findClass(String name) throws ClassNotFoundException {
        // 重写findClass方法，这是自定义类加载器的核心

        // 将包名转换为文件路径
        // 例如：com.ali.Hello -> d:\com\ali\Hello.class
        String path = this.classPath + name.replace(".", File.separator) + ".class";

        // 使用try-with-resources确保资源自动关闭
        try (FileInputStream fis = new FileInputStream(path); 
             ByteArrayOutputStream baos = new ByteArrayOutputStream()) {

            // 读取类文件的字节码
            byte[] buffer = new byte[1024];
            int len = 0;
            while ((len = fis.read(buffer)) != -1) {
                // 将读取的数据写入字节数组输出流
                baos.write(buffer, 0, len);
            }

            byte[] allByte = baos.toByteArray();  // 获取完整的字节数组

            // 调用父类的defineClass方法将字节数组转换为Class对象
            // defineClass方法会进行安全性验证、字节码验证等
            return super.defineClass(name, allByte, 0, allByte.length);

        } catch (IOException e) {
            throw new ClassNotFoundException(name + "加载失败");
        }
    }

    public static void main(String[] args) throws ClassNotFoundException, InstantiationException, IllegalAccessException {
        // 创建自定义类加载器实例，指定类文件根目录为d:\
        MyClassLoader myClassLoader = new MyClassLoader("d:\\");

        // 加载指定类
        // loadClass方法会先委托父类加载器，如果父类加载器找不到，再调用findClass方法
        Class<?> clazz = myClassLoader.loadClass("com.ali.Hello");

        // 创建类的实例（调用默认构造方法）
        clazz.newInstance();

        // 打印该类的类加载器
        System.out.println(clazz.getClassLoader()); 
        // 输出：com.MyClassLoader@哈希码
    }
}
```

**加载jar包的写法:**

```plain
从jar包加载类:String path = "jar:file:\\" + classPath + "!/" + name.replace(".", File.separator) + ".class";
```

### 2.6 注：谁来准备类加载器呢?
AppClassLoader和ExtClassLoader是Launcher的静态内部类，在程序启动时JVM会创建Launcher对象，Launcher构造器会同时会创建扩展类加载器和应用类加载器。

**Launcher类**  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1753727836124-12a08d82-6016-4a76-8a0e-7ff8b0355e1d.png)

## 3. 双亲委派机制
双亲委派机制就是: 每个类加载器都很懒，加载类时都先让父加载器去尝试加载，父加载器加载不了时自己才去加载。

> <!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1753727836464-30ba5fed-5c0f-4cc0-bdc8-026cf53f1ccd.png)
>

**例如: 加载自定义类Demo.class的流程**

1. 首先使用AppClassLoader类加载器尝试加载，AppClassLoader加载器会先检查它的缓存，查看该类是否已经被加载，有则不加载，没有则向上交给ExtClassLoader加载器。
2. ExtClassLoader加载器同样会先检查它的缓存，查看该类是否已经被加载，有则不加载，没有则向上交给Bootstrap加载器。
3. Bootstrap加载器同样会先检查它的缓存，查看该类是否已经被加载。有则不加载，没有则尝试从它负责的目录中加载，
4. Bootstrap加载器加载失败(不在它负责的目录范围)则向下交给ExtClassLoader加载器。
5. ExtClassLoader加载器会从它负责的目录中尝试加载，加载失败则向下交给AppClassLoader加载器
6. AppClassLoader加载器从它负责的classpath尝试加载，加载完成。

**双亲委派机制的好处:**

1. 避免类的重复加载：当父加载器已经加载该类时，就没有必要子加载器再加载一遍，保证被加载类的唯一性。
2. 同时Java有沙箱安全机制：自定义类的包名以 `java.`开头被禁止， 防止核心API被篡改，判断逻辑在defineClass方法中。<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1753727836275-e6ff820c-1fe1-4aa1-92f6-75582bfa5147.png)

**打破双亲委派机制**

:::color3
双亲委派机制的实现其实就是在loadClass方法中实现的。

直接调用findClass方法就可以跳过双亲委派机制,这样就可以直接加载,而不用向上委托了。

:::

```java
public static void main(String[] args) throws ClassNotFoundException, InstantiationException, IllegalAccessException {
    // 1. 创建第一个自定义类加载器实例
    //    指定类文件根目录为 d:\
    MyClassLoader myClassLoader1 = new MyClassLoader("d:\\");
    
    // 2. 直接调用findClass方法加载类
    //    注意：这里调用的是findClass()，不是loadClass()
    Class<?> clazz1 = myClassLoader1.findClass("com.ali.Hello");
    
    // 3. 打印类的哈希码
    System.out.println(clazz1.hashCode()); 
    
    // 4. 打印类的类加载器
    System.out.println(clazz1.getClassLoader()); 
}
```

```java
public static void main(String[] args) throws ClassNotFoundException, InstantiationException, IllegalAccessException {

    // 1. 创建两个不同的自定义类加载器实例
    //    注意：myClassLoader1和myClassLoader2是两个独立的对象
    MyClassLoader myClassLoader1 = new MyClassLoader("d:\\");
    MyClassLoader myClassLoader2 = new MyClassLoader("d:\\");

    // 2. 两个类加载器分别直接调用findClass方法加载同一个类
    //    这里跳过了双亲委派机制
    Class<?> clazz1 = myClassLoader1.findClass("com.ali.Hello");
    Class<?> clazz2 = myClassLoader2.findClass("com.ali.Hello");

    // 3. 打印两个Class对象的哈希码
    System.out.println(clazz1.hashCode());  // 输出不同的哈希码
    System.out.println(clazz2.hashCode());  // 输出不同的哈希码

    // 4. 打印两个Class对象的类加载器
    System.out.println(clazz1.getClassLoader());  // com.MyClassLoader@哈希码1
    System.out.println(clazz2.getClassLoader());  // com.MyClassLoader@哈希码2
}
```

## 4. ClassLoader抽象类
所有的类加载器(除了Bootstrap)都要继承ClassLoader抽象类。

**主要方法**

| 方法名 | 作用 |
| --- | --- |
| public Class<?> loadClass(String name) | 双亲委派机制的实现 |
| protected Class<?> findClass(String name) | 读取字节码文件到内存并调用defindClass方法生成Class对象 |
| protected final Class<?> defineClass(String name, byte[] b, int off, int len) | 先判断是否加载过，然后将字节数组解析成Class对象 |
| protected final void resolveClass(Class<?> c) | 连接指定的类 |


**loadClass()方法源码**

```java
protected Class<?> loadClass(String name, boolean resolve) throws ClassNotFoundException {
    synchronized (getClassLoadingLock(name)) {
        Class<?> c = findLoadedClass(name);
        if (c == null) { 
            long t0 = System.nanoTime();
            try {
                if (parent != null) {
                    c = parent.loadClass(name, false);
                } else {
                    c = findBootstrapClassOrNull(name);
                }
            } catch (ClassNotFoundException e) {

            }
            if (c == null) {
                long t1 = System.nanoTime();
                c = findClass(name);
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
}
```

## 5. URLClassLoader类
`java.net.URLClassLoader`继承了`ClassLoader类`. 拓展了功能，能够从网络或本地加载类。默认的父加载器是AppClassLoader系统类加载器。  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1753727836289-d66d241c-5df8-449b-806f-a03ac91ec6cf.png)  
**加载磁盘上的类**

```java
package classloader.urlclassloader;

public class LoadLocal {
    public LoadLocal(){
        System.out.println("本地的字节码文件");
    }
}
```

```java
package classloader.urlclassloader;

import java.io.File;
import java.net.MalformedURLException;
import java.net.URL;
import java.net.URLClassLoader;

public class Demo {
    public static void main(String[] args) throws 
                                            MalformedURLException, 
                                            ClassNotFoundException, 
                                            InstantiationException, 
                                            IllegalAccessException {
        File file = new File("D:\\Project\\java-advence\\day16-classloader\\src\\main\\java\\");
        URL url = file.toURI().toURL();
        URLClassLoader urlClassLoader = new URLClassLoader(new URL[]{url});
        Class<?> clazz = urlClassLoader.loadClass("classloader.urlclassloader.LoadLocal");
        clazz.newInstance(); 
    }
}
```

**加载网络上的类**

```java
package classloader.urlclassloader;

public class LoadURL {
    public LoadURL(){
        System.out.println("网络上的字节码文件");
    }
}
```

```java
package classloader.urlclassloader;

import java.io.File;
import java.net.MalformedURLException;
import java.net.URL;
import java.net.URLClassLoader;

public class Demo {
    public static void main(String[] args) throws 
                                            MalformedURLException, 
                                            ClassNotFoundException, 
                                            InstantiationException, 
                                            IllegalAccessException {
        URL url = new URL("http://localhost/");
        URLClassLoader urlClassLoader = new URLClassLoader(new URL[]{url});
        Class<?> clazz = urlClassLoader.loadClass("classloader.urlclassloader.LoadURL");
        clazz.newInstance(); 
    }
}
```







