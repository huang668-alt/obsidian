### 谈谈JDK8新特性。

1. Lambda表达式
2. Stream API
3. Optional类（Optional 是 Java 8 引入的一个容器类，用来优雅地表达“某个值可能为 null”的场景，避免出现 NullPointerException，同时让代码意图更清晰。）
4. 新的日期时间API
5. 默认方法（Default Methods）
6. 函数式接口和`@FunctionalInterface`
7. 方法引用（Method References）

### Java中的静态变量（static）和非静态变量有什么区别？

静态变量属于类，在类加载时分配在堆中的 Class 对象尾部，全局共享一份，生命周期与类同齐；而非静态变量属于对象，在实例创建时分配在具体对象的堆空间内，实例间相互隔离，生命周期与对象同生共死。

### 3. Java中的final关键字可以修饰什么？被final修饰后有什么特点？

**简要回答**

- **final**作为Java中的一个关键字**可以用来修饰 类 ， 方法 ，和 变量**。（但**final不能修饰构造器**）

1. **修饰类**： 被final修饰的类**不能被继承，但该类可以去继承别的 (没有被final修饰的)类**，例如String类和System类，它们被final修饰，是不可以被继承的，但是它们有自己的父类——即顶层父类Object类。 被final修饰的类虽然不能被继承，但 **可以被实例化**。
2. **修饰方法**： 被final修饰的方法**不能被子类重写，但可以被子类继承并使用（在满足访问权限规则的前提下）**。当修饰方法时， **final关键字不能与abstract关键字共存**；因为abstract修饰的方法是必须被非抽象子类重写的。
3. **修饰变量**： 被final修饰的变量称为**最终变量，即常量**——根据被修饰变量的定义位置又可分为**成员常量和局部常量**。**常量只能赋值一次，不能被二次更改**。

**详细回答**

- **final**作为Java中的一个关键字**可以用来修饰 类 ， 方法 ，和 变量**。（但**final不能修饰构造器**）

1. **修饰类**： 
	① 核心特点： 被final修饰的类**不能被继承，但该类可以去继承别的 (没有被final修饰的)类**，例如String类和System类，它们被final修饰，是不可以被继承的，但是它们有自己的父类——即顶层父类Object类。 被final修饰的类虽然不能被继承，但 **可以被实例化**。例如，**Java中所有的包装类都被final关键字修饰了**。也就是说，所有的包装类都不能被继承，但都可以被实例化。 一般地，如果**一个类已经被final关键字**修饰，那么从逻辑上讲，**该类中的方法是没有必要再次用final修饰的**。这是因为用final修饰方法的目的就是为了不让该方法被子类重写；而final修饰的类本身就已经不能被继承了，也就不可能被重写。 
	② 设计意图： 保证类的**不可变性**（如 **String** 类防止子类破坏字符串内容）。 确保**安全性**（如 **System** 类防止核心 API 被篡改）。
2. **修饰方法**： 
	① 核心特点： 被final修饰的方法**不能被子类重写，但可以被子类继承并使用（在满足访问权限规则的前提下）**。当修饰方法时， **final关键字不能与abstract关键字共存**；因为abstract修饰的方法是必须被非抽象子类重写的。 
	② 设计意图： **防止核心逻辑被修改**（如模板方法模式中的关键步骤）。
3. **修饰变量**： 
	① 核心特点： 被final修饰的变量称为**最终变量，即常量**——根据被修饰变量的定义位置又可分为**成员常量和局部常量**。**常量只能赋值一次，不能被二次更改**。 **成员常量**——成员常量必须进行初始化，可以在定义成员常量时就对它赋初值来显式初始化，若在定义成员常量时没有赋初值——需要在构造器 或者 代码块中进行初始化。 **局部常量**——局部常量如果未被使用，可以不赋初值；但如果局部常量被调用了，就必须赋初值。 **静态常量**——对于静态常量来说，只能通过在定义时为其赋初值；或者先定义，然后在静态代码块中为其赋值，这样来初始化。但是不可以在构造器中进行静态常量的初始化。 
	② 设计意图： 确保变量引用或值的**不可变性**（提升线程安全性与代码可读性）。

**知识拓展**

1. **final 修饰类**的示意图如下：

![](https://cdn.nlark.com/yuque/0/2025/jpeg/58546395/1767119201875-1be3e1d9-41dc-4da8-9889-6bfac574909a.jpeg)

2. **final 修饰属性和方法**的示意图如下：

![](https://cdn.nlark.com/yuque/0/2025/jpeg/58546395/1767119211809-016d097b-ea6b-4a74-aeb3-970daa424809.jpeg)

3. **关于 常量的命名**：
- 一般来说，常量名所有字母都大写，多个单词之间用下划线隔开。eg : MAX_VALUE（最大值）。

3. **关于 引用常量**：
- 若final关键字修饰的是一个引用类型变量，则该引用指向的地址值无法改变。（相当于一个 固定指针）

### 4. Java SE vs Java EE

- Java SE（Java Platform，Standard Edition）: Java 平台标准版，Java 编程语言的基础，它包含了支持 Java 应用程序开发和运行的核心类库以及虚拟机等核心组件。Java SE 可以用于构建桌面应用程序或简单的服务器应用程序。
- Java EE（Java Platform, Enterprise Edition ）：Java 平台企业版，建立在 Java SE 的基础上，包含了支持企业级应用程序开发和部署的标准和规范（比如 Servlet、JSP、EJB、JDBC、JPA、JTA、JavaMail、JMS）。 Java EE 可以用于构建分布式、可移植、健壮、可伸缩和安全的服务端 Java 应用程序，例如 Web 应用程序。

### 5. ⭐️JVM vs JDK vs JRE

 **JVM**：Java 虚拟机（Java Virtual Machine, JVM）是运行 Java 字节码的虚拟机。JVM 有针对不同系统的特定实现（Windows，Linux，macOS），目的是使用相同的字节码，它们都会给出相同的结果。字节码和不同系统的 JVM 实现是 Java 语言“一次编译，随处可以运行”的关键所在。

如下图所示，不同编程语言（Java、Groovy、Kotlin、JRuby、Clojure ...）通过各自的编译器编译成 `.class` 文件，并最终通过 JVM 在不同平台（Windows、Mac、Linux）上运行。

![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1766216092401-20762de0-0cd1-45ca-abd5-b8b45d28b5b5.png)

**JDK 和 JRE**
JDK（Java Development Kit）是一个功能齐全的 Java 开发工具包，供开发者使用，用于创建和编译 Java 程序。它包含了 JRE（Java Runtime Environment），以及编译器 javac 和其他工具，如 javadoc（文档生成器）、jdb（调试器）、jconsole（监控工具）、javap（反编译工具）等。

JRE 是运行已编译 Java 程序所需的环境，主要包含以下两个部分：

1. **JVM** : 也就是我们上面提到的 Java 虚拟机。
2. **Java 基础类库（Class Library）**：一组标准的类库，提供常用的功能和 API（如 I/O 操作、网络通信、数据结构等）。

下图清晰展示了 JDK、JRE 和 JVM 的关系。

![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1766216092414-d9638bf6-fff7-4d24-84dd-a290582f0915.png)

### 6. ⭐️什么是字节码?采用字节码的好处是什么?

**什么是字节码**

字节码是一种**介于源代码和机器码之间的中间代码**，它是抽象的、与具体硬件架构无关的指令集。在 Java 中，`.java`源文件经`javac`编译器编译后，生成`.class`文件，里面存储的就是 Java 字节码。

**采用字节码的好处**

- **跨平台性**：字节码不直接运行在 CPU 上，而是由 JVM（Java 虚拟机）解释或编译执行。不同操作系统安装对应版本的 JVM，就能运行相同的字节码文件，实现 “一次编写，到处运行”。
- **高效性**：相比纯解释执行源代码，字节码更接近机器码，执行效率更高；同时 JVM 还能通过 JIT（即时编译器）将热点字节码直接编译为机器码，进一步提升性能。
- **安全性**：字节码运行在 JVM 沙箱环境中，JVM 会对字节码进行校验，防止恶意代码直接操作底层系统资源。

**Java 程序从源代码到运行的过程如下图所示**：

![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1766216092420-401a24b4-37a3-4285-b5df-c870cebf0e99.png)

### 7. ⭐️为什么说 Java 语言“编译与解释并存”？。

我们可以将高级编程语言按照程序的执行方式分为两种：

- **编译型**：编译型语言 会通过编译器将源代码一次性翻译成可被该平台执行的机器码。一般情况下，编译语言的执行速度比较快，开发效率比较低。常见的编译性语言有 C、C++、Go、Rust 等等。

编译型语言（如 C、C++、Go、Rust）开发效率低的最主要原因，简单归纳就这三点：

1. **必须编译** 改一点代码就要等几秒到几分钟重新编译，而解释型语言改完直接跑，迭代速度慢很多。
2. **代码写得多** 要手动写类型声明、错误处理、样板代码，同一种功能往往比 Python/JS 多写几倍代码。
3. **调试麻烦** 错误大多在编译时一次性报出一堆，或者运行时崩溃难定位，反馈慢；解释型语言边跑边错，改起来快。

- **解释型**：解释型语言会通过解释器一句一句的将代码解释（interpret）为机器代码后再执行。解释型语言开发效率比较快，执行速度比较慢。常见的解释性语言有 Python、JavaScript、PHP 等等。

![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1766216092786-93b0762c-168d-4544-b5d1-aab1b60e5f82.png)

**为什么说 Java 语言“编译与解释并存”？**

这是因为 Java 语言既具有编译型语言的特征，也具有解释型语言的特征。因为 Java 程序要经过先编译，后解释两个步骤，由 Java 编写的程序需要先经过编译步骤，生成字节码（`.class` 文件），这种字节码必须由 Java 解释器来解释执行。

### 8. AOT 有什么优点？为什么不全部使用 AOT 呢？

JDK 9 引入了一种新的编译模式 **AOT(Ahead of Time Compilation)** 。和 JIT 不同的是，这种编译模式会**在程序被执行前就将其编译成原生机器码的可执行文件**，属于静态编译（C、 C++，Rust，Go 等语言就是静态编译。AOT 避免了 JIT 预热等各方面的开销，可以提高 Java 程序的启动速度，避免预热时间长。并且，AOT 还能减少内存占用和增强 Java 程序的安全性（AOT 编译后的代码不容易被反编译和修改），特别适合云原生场景。

**JIT 与 AOT 两者的关键指标对比**:

![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1766216092942-bd6f50f3-537c-47d1-8654-5bbce1f9a041.png)

AOT 的主要优势在于启动时间、内存占用和打包体积。JIT 的主要优势在于具备更高的极限处理能力，可以降低请求的最大延迟。

**AOT（Ahead-of-Time）的主要优势：**

- **启动时间更快**：AOT 在构建阶段提前将模板和组件编译成高效的 JavaScript 代码，浏览器无需在运行时编译，直接执行预编译代码，避免了 JIT 的运行时编译开销。
- **内存占用更低**：无需在运行时加载和运行 Angular 编译器，也无需存储中间表示或 profiling 数据，整体运行时内存足迹更小。
- **打包体积更小**：AOT 不需要将 Angular 编译器打包到生产 bundle 中（JIT 需要），同时模板内联优化进一步减少了文件大小。

**JIT（Just-in-Time）的主要优势：**

- **更高的极限处理能力（峰值性能）**：JIT 在运行时基于实际执行路径和 profiling 数据进行动态优化（如激进内联、分支预测、专有硬件优化），能生成更针对性的机器码，通常在长时间运行后达到高于 AOT 的峰值性能。
- **降低请求的最大延迟**：经过 warmup 后，JIT 的动态优化使热路径代码执行更快、更稳定，在高负载或长运行服务中，尾延迟（p99/p999 延迟）更低，而 AOT 的性能更固定（无运行时适应）。

提到 AOT 就不得不提 GraalVM 了！GraalVM 是一种高性能的 JDK（完整的 JDK 发行版本），它可以运行 Java 和其他 JVM 语言，以及 JavaScript、Python 等非 JVM 语言。 GraalVM 不仅能提供 AOT 编译，还能提供 JIT 编译。感兴趣的同学，可以去看看 GraalVM 的官方文档：[https://www.graalvm.org/latest/docs/](https://www.graalvm.org/latest/docs/)。如果觉得官方文档看着比较难理解的话，也可以找一些文章来看看，比如：

- [基于静态编译构建微服务应用](https://mp.weixin.qq.com/s/4haTyXUmh8m-dBQaEzwDJw)
- [走向 Native 化：Spring&Dubbo AOT 技术示例与原理讲解](https://cn.dubbo.apache.org/zh-cn/blog/2023/06/28/%e8%b5%b0%e5%90%91-native-%e5%8c%96springdubbo-aot-%e6%8a%80%e6%9c%af%e7%a4%ba%e4%be%8b%e4%b8%8e%e5%8e%9f%e7%90%86%e8%ae%b2%e8%a7%a3/)

**既然 AOT 这么多优点，那为什么不全部使用这种编译方式呢？**

我们前面也对比过 JIT 与 AOT，两者各有优点，只能说 AOT 更适合当下的云原生场景，对微服务架构的支持也比较友好。**除此之外，AOT 编译无法支持 Java 的一些动态特性，如反射、动态代理、动态加载、JNI（Java Native Interface）等。然而，很多框架和库（如 Spring、CGLIB）都用到了这些特性。如果只使用 AOT 编译，那就没办法使用这些框架和库了，或者说需要针对性地去做适配和优化。**

### 9. Java 和 C++ 的区别?

Java 和 C++ 都是面向对象的语言，都支持封装、继承和多态，但是，它们还是有挺多不相同的地方：

- Java 不提供指针来直接访问内存，程序内存更加安全
- Java 的类是单继承的，C++ 支持多重继承；虽然 Java 的类不可以多继承，但是接口可以多继承。
- Java 有自动内存管理垃圾回收机制(GC)，不需要程序员手动释放无用内存。
- C ++同时支持方法重载和操作符重载，但是 Java 只支持方法重载（操作符重载增加了复杂性，这与 Java 最初的设计思想不符）。

### 10. 注释有哪几种形式？

Java 中的注释有三种：

1. **单行注释**：通常用于解释方法内某单行代码的作用。
2. **多行注释**：通常用于解释一段代码的作用。
3. **文档注释**：通常用于生成 Java 开发文档。

### 11. 标识符和关键字的区别是什么？

在我们编写程序的时候，需要大量地为程序、类、变量、方法等取名字，于是就有了 **标识符** 。简单来说， **标识符就是一个名字** 。

有一些标识符，Java 语言已经赋予了其特殊的含义，只能用于特定的地方，这些特殊的标识符就是 **关键字** 。简单来说，**关键字是被赋予特殊含义的标识符** 。

### 12. Java 语言关键字有哪些？

| 分类         | 关键字      |            |          |              |            |           |        |
| ---------- | -------- | ---------- | -------- | ------------ | ---------- | --------- | ------ |
| 访问控制       | private  | protected  | public   |              |            |           |        |
| 类，方法和变量修饰符 | abstract | class      | extends  | final        | implements | interface | native |
|            | new      | static     | strictfp | synchronized | transient  | volatile  | enum   |
| 程序控制       | break    | continue   | return   | do           | while      | if        | else   |
|            | for      | instanceof | switch   | case         | default    | assert    |        |
| 错误处理       | try      | catch      | throw    | throws       | finally    |           |        |
| 包相关        | import   | package    |          |              |            |           |        |
| 基本类型       | boolean  | byte       | char     | double       | float      | int       | long   |
|            | short    |            |          |              |            |           |        |
| 变量引用       | super    | this       | void     |              |            |           |        |
| 保留字        | goto     | const      |          |              |            |           |        |

Tips：所有的关键字都是小写的，在 IDE 中会以特殊颜色显示。

`default` 这个关键字很特殊，既属于程序控制，也属于类，方法和变量修饰符，还属于访问控制。

- 在程序控制中，当在 `switch` 中匹配不到任何情况时，可以使用 `default` 来编写默认匹配的情况。
- 在类，方法和变量修饰符中，从 JDK8 开始引入了默认方法，可以使用 `default` 关键字来定义一个方法的默认实现。
- 在访问控制中，如果一个方法前没有任何修饰符，则默认会有一个修饰符 `default`，但是这个修饰符加上了就会报错。

⚠️ 注意：虽然 `true`, `false`, 和 `null` 看起来像关键字但实际上他们是字面值，同时你也不可以作为标识符来使用。

### 15. continue、break 和 return 的区别是什么？

在循环结构中，当循环条件不满足或者循环次数达到要求时，循环会正常结束。但是，有时候可能需要在循环的过程中，当发生了某种条件之后 ，提前终止循环，这就需要用到下面几个关键词：

1. `continue`：指跳出当前的这一次循环，继续下一次循环。
	
2. `break`：指跳出整个循环体，继续执行循环下面的语句。
	
3. `return` 用于跳出所在方法，结束该方法的运行。return 一般有两种用法：
	1. `return;`：直接使用 return 结束方法执行，用于没有返回值函数的方法
	2. `return value;`：return 一个特定值，用于有返回值函数的方法

### 16. Java 中的几种基本数据类型了解么？

| 基本类型      | 位数  | 字节  | 默认值     | 取值范围                                                       |
| --------- | --- | --- | ------- | ---------------------------------------------------------- |
| `byte`    | 8   | 1   | 0       | -128 ~ 127                                                 |
| `short`   | 16  | 2   | 0       | -32768（-2^15） ~ 32767（2^15 - 1）                            |
| `int`     | 32  | 4   | 0       | -2147483648 ~ 2147483647                                   |
| `long`    | 64  | 8   | 0L      | -9223372036854775808（-2^63） ~ 9223372036854775807（2^63 -1） |
| `char`    | 16  | 2   | 'u0000' | 0 ~ 65535（2^16 - 1）                                        |
| `float`   | 32  | 4   | 0f      | 1.4E-45 ~ 3.4028235E38                                     |
| `double`  | 64  | 8   | 0d      | 4.9E-324 ~ 1.7976931348623157E308                          |
| `boolean` | 1   |     | false   | true、false                                                 |
 `byte`、`short`、`int`、`long`能表示的最大正数都减 1 了
因为在二进制补码表示法中，最高位是用来表示符号的（0 表示正数，1 表示负数），其余位表示数值部分。所以，如果我们要表示最大的正数，我们需要把除了最高位之外的所有位都设为 1。如果我们再加 1，就会导致溢出，变成一个负数。

对于 `boolean`，官方文档未明确定义，它依赖于 JVM 厂商的具体实现。逻辑上理解是占用 1 位，但是实际中会考虑计算机高效存储因素。

另外，Java 的每种基本类型所占存储空间的大小不会像其他大多数语言那样随机器硬件架构的变化而变化。这种所占存储空间大小的不变性是 Java 程序比用其他大多数语言编写的程序更具可移植性的原因之一。

**注意：**

1. Java 里使用 `long` 类型的数据一定要在数值后面加上 **L**，否则将作为整型解析。
2. Java 里使用 `float` 类型的数据一定要在数值后面加上 **f 或 F**，否则将无法通过编译。
3. `char a = 'h'`char :单引号，`String a = "hello"` :双引号。
4. 这八种基本类型都有对应的包装类分别为：`Byte`、`Short`、`Integer`、`Long`、`Float`、`Double`、`Character`、`Boolean` 。

### 17. 基本类型和包装类型的区别？

- **用途与泛型**：基本类型多用于局部变量和常量，而包装类型是正牌对象。因为 Java 泛型存在**编译期类型擦除**，要求必须是 Object 及其子类，所以**泛型只能用包装类**。
    
- **内存存储与占用**：基本类型的局部变量在**栈帧的局部变量表**中，成员变量在**堆中的对象体内**，空间开销极小。而包装类型属于对象，必须在**堆中**开辟完整的对象结构（包含对象头、对齐填充等），**内存占用远大于基本类型**。
    
    _(注：静态基本类型变量，现代 JVM 是存在堆中的 Class 对象尾部，而非元空间。)_
    
- **初始默认值**：基本类型有明确的底层默认值（如 `0`, `false`）；而包装类属于引用类型，未显式初始化前**一律为 `null`**，这在业务架构（如 RPC 接口、数据库实体映射）中能精准区分‘未赋值’和‘真的为 0’。
    
- **比较方式的底层差异**：基本类型用 `==` 直接比对**栈里的字面物理数值**。包装类型用 `==` 则是比对**堆内存的物理地址**。
    
- **核心避坑指南**：虽然整型包装类（如 `Integer`）存在 $-128 \sim 127$ 之间的**常量池缓存机制（IntegerCache）导致小整数用 `==` 偶尔能跑通，但为了避免大数发生地址撕裂，所有包装类的数值比较，在生产环境必须全部强制使用 `equals()` 方法。**”

### 18. 包装类型的缓存机制了解么？

Java 基本数据类型的包装类型的大部分都用到了缓存机制来提升性能。

`Byte`,`Short`,`Integer`,`Long` 这 4 种包装类默认创建了数值 **[-128，127]** 的相应类型的缓存数据，`Character` 创建了数值在 **[0,127]** 范围的缓存数据，`Boolean` 直接返回 `TRUE` or `FALSE`。

对于 `Integer`，可以通过 JVM 参数 `-XX:AutoBoxCacheMax=<size>` 修改缓存上限，但不能修改下限 -128。实际使用时，并不建议设置过大的值，避免浪费内存，甚至是 OOM。

对于`Byte`,`Short`,`Long` ,`Character` 没有类似 `-XX:AutoBoxCacheMax` 参数可以修改，因此缓存范围是固定的，无法通过 JVM 参数调整。`Boolean` 则直接返回预定义的 `TRUE` 和 `FALSE` 实例，没有缓存范围的概念。

**Integer 缓存源码：**

``` Java
public static Integer valueOf(int i) {
    // 如果传入的 int 值 i 在缓存范围之内（默认 -128 ~ 127）
    if (i >= IntegerCache.low && i <= IntegerCache.high)
        // 直接返回缓存中的 Integer 对象
        // cache 数组的下标通过偏移计算：i + (-low) = i + 128
        // 例如 i = -128 → 下标 0；i = 0 → 下标 128；i = 127 → 下标 255
        return IntegerCache.cache[i + (-IntegerCache.low)];
    
    // 如果超出缓存范围，则新建一个 Integer 对象（不再复用）
    return new Integer(i);
}

private static class IntegerCache {
    // 缓存的下界，固定为 -128（不可配置）
    static final int low = -128;
    
    // 缓存的上界，默认 127，但可以通过 JVM 参数修改
    static final int high;
    
    // 缓存数组，存放提前创建好的 Integer 对象
    static final Integer[] cache;
    
    // 静态初始化块：在类加载时执行一次
    static {
        // 默认上界为 127
        int h = 127;
        
        // 支持通过系统属性配置更大的缓存上界
        // JVM 参数：-XX:AutoBoxCacheMax=<size>（部分 JDK 支持）
        // 或者更常见的：-Djava.lang.Integer.IntegerCache.high=<value>
        String integerCacheHighPropValue =
            VM.getSavedProperty("java.lang.Integer.IntegerCache.high");
        if (integerCacheHighPropValue != null) {
            try {
                int i = Integer.parseInt(integerCacheHighPropValue);
                // 取较大值（但不能低于 127）
                i = Math.max(i, 127);
                h = i;
            } catch (NumberFormatException nfe) {
                // 如果属性值非法，就忽略，使用默认 127
            }
        }
        high = h;  // 设置最终的 high 值
        
        // 计算缓存数组大小：从 low 到 high 共 (high - low + 1) 个元素
        int size = (high - low) + 1;
        
        // 创建数组并填充提前 new 好的 Integer 对象
        cache = new Integer[size];
        int j = low;  // 从 -128 开始
        for (int k = 0; k < cache.length; k++) {
            cache[k] = new Integer(j++);
        }
        
        // 断言：确保 high 至少是 127（JDK 内部安全检查）
        assert high >= 127;
    }
    
    // 私有构造器，防止外部实例化 IntegerCache
    private IntegerCache() {}
}
```

`Character` **缓存源码:**

``` Java
public static Character valueOf(char c) {
    // 如果传入的 char 值 c <= 127（即 ASCII 范围 0~127）
    // 这个范围的 Character 对象必须缓存（官方要求）
    if (c <= 127) { 
        // 直接返回预先创建好的缓存对象
        // cache 数组大小为 128（索引 0~127），直接用 (int)c 作为下标
        // 例如：c = 'A' (65) → 返回 cache[65]
        return CharacterCache.cache[(int)c];
    }
    // 如果 c > 127（例如中文、emoji 等 Unicode 字符）
    // 则每次都新建一个 Character 对象，不缓存
    return new Character(c);
}

private static class CharacterCache {
    // 私有构造器，防止外部实例化这个缓存类
    private CharacterCache() {}

    // 缓存数组：固定大小 128，存放 char 值从 0 到 127 对应的 Character 对象
    static final Character[] cache = new Character[127 + 1];

    // 静态初始化块：在 Character 类加载时执行一次
    static {
        // 遍历 0~127，提前 new 出所有可能的 Character 对象并放入缓存
        for (int i = 0; i < cache.length; i++) {
            cache[i] = new Character((char)i);
        }
    }
}
```

`Boolean` **缓存源码：**

``` Java
public static Boolean valueOf(boolean b) {
    return (b ? TRUE : FALSE);
}
```

如果超出对应范围仍然会去创建新的对象，缓存的范围区间的大小只是在性能和资源之间的权衡。

两种浮点数类型的包装类 `Float`,`Double` 并没有实现缓存机制。

``` Java
Integer i1 = 33;
Integer i2 = 33;
System.out.println(i1 == i2);// 输出 true

Float i11 = 333f;
Float i22 = 333f;
System.out.println(i11 == i22);// 输出 false

Double i3 = 1.2;
Double i4 = 1.2;
System.out.println(i3 == i4);// 输出 false
```

下面我们来看一个问题：下面的代码的输出结果是 `true` 还是 `false` 呢？

``` Java
Integer i1 = 40;
Integer i2 = new Integer(40);
System.out.println(i1==i2);
```

`Integer i1=40` 这一行代码会发生装箱，也就是说这行代码等价于 `Integer i1=Integer.valueOf(40)` 。因此，`i1` 直接使用的是缓存中的对象。而`Integer i2 = new Integer(40)` 会直接创建新的对象。

因此，答案是 `false` 。你答对了吗？

记住：**所有整型包装类对象之间值的比较，全部使用 equals 方法比较**。

![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1766216093145-153e5915-73d2-40d8-80b1-95a4e4cc75d5.png)

### 19. 自动装箱与拆箱了解吗？原理是什么？

**什么是自动拆装箱？**

- **装箱**：将基本类型用它们对应的引用类型包装起来；
- **拆箱**：将包装类型转换为基本数据类型；

举例：

``` Java
Integer i = 10;  //装箱
int n = i;   //拆箱
```

上面这两行代码对应的字节码为：

``` 
L1
    LINENUMBER 8 L1                  // 对应源代码第8行
    ALOAD 0                          // 把 this（当前对象实例）加载到操作数栈顶
                                     // （因为 i 是实例字段，需要通过对象访问）

    BIPUSH 10                        // 把常量 10（byte 类型推送）推到操作数栈顶
                                     // BIPUSH 用于 -128~127 的小整数常量

    INVOKESTATIC java/lang/Integer.valueOf (I)Ljava/lang/Integer;
                                     // 调用 Integer.valueOf(int) 方法
                                     // 消耗栈顶的 int 10，返回一个 Integer 对象
                                     // 这就是“自动装箱”的实现：int → Integer

    PUTFIELD AutoBoxTest.i : Ljava/lang/Integer;
                                     // 把栈顶的 Integer 对象赋值给当前对象的 i 字段
                                     // 完成 i = 10;（装箱赋值）

L2
    LINENUMBER 9 L2                  // 对应源代码第9行
    ALOAD 0                          // 再次加载 this 到栈顶（需要访问 i 和 n 字段）

    ALOAD 0                          // 再次加载 this（为了获取 i 字段的值）
    
    GETFIELD AutoBoxTest.i : Ljava/lang/Integer;
                                     // 从当前对象取出 i 字段（Integer 对象）推到栈顶

    INVOKEVIRTUAL java/lang/Integer.intValue ()I
                                     // 调用 Integer 实例的 intValue() 方法
                                     // 消耗栈顶的 Integer 对象，返回 int 值
                                     // 这就是“自动拆箱”的实现：Integer → int

    PUTFIELD AutoBoxTest.n : I
                                     // 把栈顶的 int 值赋值给当前对象的 n 字段
                                     // 完成 n = i;（拆箱赋值）

    RETURN                           // 方法返回（void 方法）
```

从字节码中，我们发现装箱其实就是调用了 包装类的`valueOf()`方法，拆箱其实就是调用了 `xxxValue()`方法。

因此，

- `Integer i = 10` 等价于 `Integer i = Integer.valueOf(10)`
- `int n = i` 等价于 `int n = i.intValue()`;

注意：**如果频繁拆装箱的话，也会严重影响系统的性能。我们应该尽量避免不必要的拆装箱操作。**

``` Java
private static long sum() {
    // 应该使用 long 而不是 Long
    Long sum = 0L;
    for (long i = 0; i <= Integer.MAX_VALUE; i++)
    sum += i;
    return sum;
}
```

**日常开发中 90% 的场景都应该使用基本数据类型，只有在必须使用对象特性（null、泛型、集合、对象方法）时才使用包装类型。**

这样写出的代码：

- 更快（性能更好）
- 更省内存（GC 压力小）
- 更安全（避免不必要的 NPE）
- 更清晰（默认值明确，语义明确）

### 20. 为什么浮点数运算的时候会有精度丢失的风险？

浮点数运算精度丢失代码演示：

``` Java
float a = 2.0f - 1.9f;
float b = 1.8f - 1.7f;
System.out.printf("%.9f",a);// 0.100000024
System.out.println(b);// 0.099999905
System.out.println(a == b);// false
```

这个和计算机保存浮点数的机制有很大关系。我们知道计算机是二进制的，而且计算机在表示一个数字时，宽度是有限的，无限循环的小数存储在计算机时，只能被截断，所以就会导致小数精度发生损失的情况。这也就是解释了为什么浮点数没有办法用二进制精确表示。

就比如说十进制下的 0.2 就没办法精确转换成二进制小数：

```
// 0.2 转换为二进制数的过程为，不断乘以 2，直到不存在小数为止，
// 在这个计算过程中，得到的整数部分从上到下排列就是二进制的结果。
0.2 * 2 = 0.4 -> 0
0.4 * 2 = 0.8 -> 0
0.8 * 2 = 1.6 -> 1
0.6 * 2 = 1.2 -> 1
0.2 * 2 = 0.4 -> 0（发生循环）
...
```

关于浮点数的更多内容，建议看一下[[计算机系统基础（四）浮点数]]这篇文章。

### 21. 如何解决浮点数运算的精度丢失问题？

`BigDecimal` 可以实现对浮点数的运算，不会造成精度丢失。通常情况下，大部分需要浮点数精确运算结果的业务场景（比如涉及到钱的场景）都是通过 `BigDecimal` 来做的。

``` Java
BigDecimal a = new BigDecimal("1.0");
BigDecimal b = new BigDecimal("1.00");
BigDecimal c = new BigDecimal("0.8");

BigDecimal x = a.subtract(c);
BigDecimal y = b.subtract(c);

System.out.println(x); /* 0.2 */
System.out.println(y); /* 0.20 */
// 比较内容，不是比较值
System.out.println(Objects.equals(x, y)); /* false */
// 比较值相等用相等compareTo，相等返回0
System.out.println(0 == x.compareTo(y)); /* true */
```

关于 `BigDecimal` 的详细介绍，可以看看我写的这篇文章：[[BigDecimal 详解]]
### 22. 超过 long 整型的数据应该如何表示？

 `java.math.BigInteger`（官方正统解法）

当整数范围超过 `long` 时，官方的标准答案就是 `BigInteger`。**它的理论支持范围是无限大**（只受限于你计算机实际的内存大小）。

``` Java
// 1. 创建：不能直接用字面量赋值，必须传入字符串或通过静态方法
BigInteger bigNum1 = new BigInteger("9223372036854775808"); // 超过 long 最大值 1
BigInteger bigNum2 = BigInteger.valueOf(100L); // 如果数字没超 long，可以用 valueOf

// 2. 运算：因为是对象，不能使用 +, -, *, /，必须使用对应的方法
BigInteger result = bigNum1.add(bigNum2); // 加法
```

 **🧠 BigInteger 的物理底层是如何实现的？**

很多面试官喜欢追问：“为什么 `BigInteger` 能存无限大的数字？它的底层结构是什么？”

在底层，`BigInteger` 抛弃了计算机硬件级对 64 位整型的直接限制，改用了一个动态扩容的整型数组（`int[]`）来人肉拼凑大数。我们去看它的源码：

``` Java
public class BigInteger extends Number implements Comparable<BigInteger> {
    final int signum; // 符号位：1位正数，-1位负数，0位零
    final int[] mag;  // 核心：用来存储绝对值二进制大小的 int 数组
}
```

- **大数拆分**：一个 `int` 占用 32 位。如果你输入的数字极大，JVM 会把这个数字转换成二进制，然后**每 32 位切一刀**，像一节一节车厢一样，按顺序塞进 `mag` 数组里。
    
- **计算模拟**：当你调用 `.add()` 或 `.multiply()` 时，JVM 无法依赖 CPU 的底层加法器直接计算，而是通过底层的 C++/Java 代码，**手动模拟我们小学学过的竖式加法和乘法**，一位一位地计算并处理进位（Carry）。因此，它的**计算性能比基本类型慢得多**，但换来了无限的精度。

### 23. ⭐️成员变量与局部变量的区别？

- **语法形式**：从语法形式上看，成员变量是属于类的，而局部变量是在代码块或方法中定义的变量或是方法的参数；成员变量可以被 `public`,`private`,`static` 等修饰符所修饰，而局部变量不能被访问控制修饰符及 `static` 所修饰；但是，成员变量和局部变量都能被 `final` 所修饰。
- **存储方式**：从变量在内存中的存储方式来看，如果成员变量是使用 `static` 修饰的，那么这个成员变量是属于类的，如果没有使用 `static` 修饰，这个成员变量是属于实例的。而对象存在于堆内存，局部变量则存在于栈内存。
- **生存时间**：从变量在内存中的生存时间上看，成员变量是对象的一部分，它随着对象的创建而存在，而局部变量随着方法的调用而自动生成，随着方法的调用结束而消亡。
- **默认值**：从变量是否有默认值来看，成员变量如果没有被赋初始值，则会自动以类型的默认值而赋值（一种情况例外:被 `final` 修饰的成员变量也必须显式地赋值），而局部变量则不会自动赋值。

### **为什么成员变量有默认值？**

“**成员变量有默认值是为了保障 JVM 的内存安全。** 因为对象创建在堆中，申请到的物理内存可能残留有其他进程的脏数据，JVM 在分配空间后会强制将其物理清零（也就是赋予默认值），防止程序读取到流氓指针或垃圾数据。

相反，**局部变量没有默认值是为了追求极致的执行效率。** 局部变量存在于高频创建销毁的栈帧中，如果由 JVM 在运行期动态清零会严重拖慢速度，因此 Java 编译器选择通过静态数据流分析，在编译期强制要求程序员必须手动初始化局部变量。”

### 24. 静态变量有什么作用？

- **实现数据共享**：将变量与具体的对象个体剥离，升级到‘类’的高度，供所有实例全局共享、统一读写；
    
- **优化内存开销**：由于它在内存中仅此一份（随类加载诞生在堆的 Class 对象尾部），非常适合配合 `final` 组合定义全局常量，避免大量 new 对象造成的垃圾碎片；
    
- **作为架构支撑**：它是类加载机制天然同步的产物，是实现高并发单例模式、全局工具箱以及高性能缓存的底层骨架。”

### 25. 字符型常量和字符串常量的区别?

- **形式** : 字符常量是单引号引起的一个字符，字符串常量是双引号引起的 0 个或若干个字符。
- **含义** : 字符常量相当于一个整型值( ASCII 值),可以参加表达式运算; 字符串常量代表一个地址值(该字符串在内存中存放位置)。
- **占内存大小**：字符常量只占 2 个字节; 字符串常量占若干个字节。

⚠️ 注意 `char` 在 Java 中占两个字节。

字符型常量和字符串常量代码示例：

``` Java
public class StringExample {
    // 字符型常量
    public static final char LETTER_A = 'A';

    // 字符串常量
    public static final String GREETING_MESSAGE = "Hello, world!";
    public static void main(String[] args) {
        System.out.println("字符型常量占用的字节数为："+Character.BYTES);
        System.out.println("字符串常量占用的字节数为："+GREETING_MESSAGE.getBytes().length);
    }
}
```

输出：

```
字符型常量占用的字节数为：2
字符串常量占用的字节数为：13
```

### 26. 什么是方法的返回值?方法有哪几种类型？

**方法的返回值** 是指我们获取到的某个方法体中的代码执行后产生的结果！（前提是该方法可能产生结果）。返回值的作用是接收出结果，使得它可以用于其他的操作！

我们可以按照方法的返回值和参数类型将方法分为下面这几种：

**1、无参数无返回值的方法**

``` Java
public void f1() {
    //......
}
// 下面这个方法也没有返回值，虽然用到了 return
public void f(int a) {
    if (...) {
        // 表示结束方法的执行,下方的输出语句不会执行
        return;
    }
    System.out.println(a);
}
```

**2、有参数无返回值的方法**

``` Java
public void f2(Parameter 1, ..., Parameter n) {
    //......
}
```

**3、有返回值无参数的方法**

``` Java
public int f3() {
    //......
    return x;
}
```

**4、有返回值有参数的方法**

``` Java
public int f4(int a, int b) {
    return a * b;
}
```

### 27. 静态方法为什么不能调用非静态成员?

1. 静态方法是属于类的，在类加载的时候就会分配内存，可以通过类名直接访问。而非静态成员属于实例对象，只有在对象实例化之后才存在，需要通过类的实例对象去访问。
2. 在类的非静态成员不存在的时候静态方法就已经存在了，此时调用在内存中还不存在的非静态成员，属于非法操作。

### 28. ⭐️静态方法和实例方法有何不同？

**1、调用方式**

在外部调用静态方法时，可以使用 `类名.方法名` 的方式，也可以使用 `对象.方法名` 的方式，而实例方法只有后面这种方式。也就是说，**调用静态方法可以无需创建对象** 。

不过，需要注意的是一般不建议使用 `对象.方法名` 的方式来调用静态方法。这种方式非常容易造成混淆，静态方法不属于类的某个对象而是属于这个类。

因此，一般建议使用 `类名.方法名` 的方式来调用静态方法。

**2、访问类成员是否存在限制**

静态方法在访问本类的成员时，只允许访问静态成员（即静态成员变量和静态方法），不允许访问实例成员（即实例成员变量和实例方法），而实例方法不存在这个限制。

### 29. ⭐️重载和重写有什么区别？

重载就是同样的一个方法能够根据输入数据的不同，做出不同的处理

重写就是当子类继承自父类的相同方法，输入数据一样，但要做出有别于父类的响应时，你就要覆盖父类方法

**重载**

发生在同一个类中（或者父类和子类之间），方法名必须相同，参数类型不同、个数不同、顺序不同，方法返回值和访问修饰符可以不同。

《Java 核心技术》这本书是这样介绍重载的：

如果多个方法(比如 `StringBuilder` 的构造方法)有相同的名字、不同的参数， 便产生了重载。

编译器必须挑选出具体执行哪个方法，它通过用各个方法给出的参数类型与特定方法调用所使用的值类型进行匹配来挑选出相应的方法。 如果编译器找不到匹配的参数， 就会产生编译时错误， 因为根本不存在匹配， 或者没有一个比其他的更好(这个过程被称为重载解析(overloading resolution))。

Java 允许重载任何方法， 而不只是构造器方法。

``` Java
StringBuilder sb = new StringBuilder();
StringBuilder sb2 = new StringBuilder("HelloWorld");
```

综上：重载就是同一个类中多个同名方法根据不同的传参来执行不同的逻辑处理。

**重写**

重写发生在运行期，是子类对父类的允许访问的方法的实现过程进行重新编写。

1. 方法名、参数列表必须相同，子类方法返回值类型应比父类方法返回值类型更小或相等，抛出的异常范围小于等于父类，访问修饰符范围大于等于父类。
2. 如果父类方法访问修饰符为 `private/final/static` 则子类就不能重写该方法，但是被 `static` 修饰的方法能够被再次声明。
3. 构造方法无法被重写

#### 总结

综上：**重写就是子类对父类方法的重新改造，外部样子不能改变，内部逻辑可以改变。**

| 区别点       | 重载 (Overloading)                               | 重写 (Overriding)                                                     |
| --------- | ---------------------------------------------- | ------------------------------------------------------------------- |
| **发生范围**  | 同一个类中。                                         | 父类与子类之间（存在继承关系）。                                                    |
| **方法签名**  | 方法名**必须相同**，但**参数列表必须不同**（参数的类型、个数或顺序至少有一项不同）。 | 方法名、参数列表**必须完全相同**。                                                 |
| **返回类型**  | 与返回值类型**无关**，可以任意修改。                           | 子类方法的返回类型必须与父类方法的返回类型**相同**，或者是其**子类**。                             |
| **访问修饰符** | 与访问修饰符**无关**，可以任意修改。                           | 子类方法的访问权限**不能低于**父类方法的访问权限。（public > protected > default > private） |
| **绑定时期**  | 编译时绑定或称静态绑定                                    | 运行时绑定 (Run-time Binding) 或称动态绑定                                     |

⭐️ 关于 **重写的返回值类型** 这里需要额外多说明一下，上面的表述不太清晰准确：如果方法的返回类型是 void 和基本数据类型，则返回值重写时不可修改。但是如果方法的返回值是引用类型，重写时是可以返回该引用类型的子类的。

``` Java
public class Hero {
    public String name() {
        return "超级英雄";
    }
}
public class SuperMan extends Hero{
    @Override
    public String name() {
        return "超人";
    }
    public Hero hero() {
        return new Hero();
    }
}

public class SuperSuperMan extends SuperMan {
    @Override
    public String name() {
        return "超级超级英雄";
    }

    @Override
    public SuperMan hero() {
        return new SuperMan();
    }
}
```

### 30. 什么是可变长参数？

从 Java5 开始，Java 支持定义可变长参数，所谓可变长参数就是允许在调用方法时传入不定长度的参数。就比如下面这个方法就可以接受 0 个或者多个参数。

``` Java
public static void method1(String... args) {
    //......
}
```

另外，可变参数只能作为函数的最后一个参数，但其前面可以有也可以没有任何其他参数。

``` Java
public static void method2(String arg1, String... args) {
    //......
}
// 最少需要 1 个参数
```

**遇到方法重载的情况怎么办呢？会优先匹配固定参数还是可变参数的方法呢？**

会优先匹配固定参数的方法，因为固定参数的方法匹配度更高。

### 31. ⭐️面向对象和面向过程的区别

面向过程编程（Procedural-Oriented Programming，POP）和面向对象编程（Object-Oriented Programming，OOP）是两种常见的编程范式，两者的主要区别在于解决问题的方式不同：

- **面向过程编程（POP）**：面向过程把解决问题的过程拆成一个个方法，通过一个个方法的执行解决问题。
- **面向对象编程（OOP）**：面向对象会先抽象出对象，然后用对象执行方法的方式解决问题。

相比较于 POP，OOP 开发的程序一般具有下面这些优点：

- **易维护**：由于良好的结构和封装性，OOP 程序通常更容易维护。
- **易复用**：通过继承和多态，OOP 设计使得代码更具复用性，方便扩展功能。
- **易扩展**：模块化设计使得系统扩展变得更加容易和灵活。

POP 的编程方式通常更为简单和直接，适合处理一些较简单的任务。

POP 和 OOP 的性能差异主要取决于它们的运行机制，而不仅仅是编程范式本身。因此，简单地比较两者的性能是一个常见的误区（相关 issue : [面向过程：面向过程性能比面向对象高？？](https://github.com/Snailclimb/JavaGuide/issues/431) ）。

![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1766238302206-1eb37e13-0bad-4e29-90b1-57a597a7501d.png?x-oss-process=image%2Fcrop%2Cx_0%2Cy_0%2Cw_1330%2Ch_464)

在选择编程范式时，性能并不是唯一的考虑因素。代码的可维护性、可扩展性和开发效率同样重要。

现代编程语言基本都支持多种编程范式，既可以用来进行面向过程编程，也可以进行面向对象编程。

下面是一个求圆的面积和周长的示例，简单分别展示了面向对象和面向过程两种不同的解决方案。

**面向对象**：

```
public class Circle {
    // 定义圆的半径
    private double radius;

    // 构造函数
    public Circle(double radius) {
        this.radius = radius;
    }

    // 计算圆的面积
    public double getArea() {
        return Math.PI * radius * radius;
    }

    // 计算圆的周长
    public double getPerimeter() {
        return 2 * Math.PI * radius;
    }

    public static void main(String[] args) {
        // 创建一个半径为3的圆
        Circle circle = new Circle(3.0);

        // 输出圆的面积和周长
        System.out.println("圆的面积为：" + circle.getArea());
        System.out.println("圆的周长为：" + circle.getPerimeter());
    }
}
```

我们定义了一个 `Circle` 类来表示圆，该类包含了圆的半径属性和计算面积、周长的方法。

**面向过程**：

```
public class Main {
    public static void main(String[] args) {
        // 定义圆的半径
        double radius = 3.0;

        // 计算圆的面积和周长
        double area = Math.PI * radius * radius;
        double perimeter = 2 * Math.PI * radius;

        // 输出圆的面积和周长
        System.out.println("圆的面积为：" + area);
        System.out.println("圆的周长为：" + perimeter);
    }
}
```

我们直接定义了圆的半径，并使用该半径直接计算出圆的面积和周长。

### 32. 创建一个对象用什么运算符?对象实体与对象引用有何不同?

**创建对象的运算符**

创建一个对象使用的是 **`new`** 运算符。它负责在堆内存中为对象分配空间，并触发构造方法完成初始化。

**对象实体 vs 对象引用**

最直观的比喻是：**对象实体是“电视机”，对象引用是“遥控器”。**

它们的核心区别有以下三点：

- **本质与内存位置不同：**
    
    - **对象实体**是真正的实例，存放在堆内存（Heap）中，包含具体的数据和方法，占用较大内存。
        
    - **对象引用**是一个变量，存放在**栈内存（Stack）中，里面存储的是对象实体在堆中的**内存首地址。
        
- **对应关系不同：**
    
    - 一个引用在同一时刻只能指向 **0 个或 1 个**对象实体（可以为 `null`，也可以换台）。
        
    - 一个对象实体可以同时被 **多个** 引用指向（多个人拿遥控器控制同一台电视）。
        
- **生命周期不同：**
    
    - 引用随着作用域的结束而从栈中销毁。
        
    - 实体在失去所有引用指向后，成为“内存垃圾”，等待 JVM 的 **垃圾回收器（GC）** 异步择机回收。

### 33. ⭐️对象的相等和引用相等的区别

- 对象的相等一般比较的是内存中存放的内容是否相等。

- 引用相等一般比较的是他们指向的内存地址是否相等。

### 34. 如果一个类没有声明构造方法，该程序能正确执行吗?

构造方法是一种特殊的方法，主要作用是完成对象的初始化工作。

如果一个类没有声明构造方法，也可以执行！因为一个类即使没有声明构造方法也会有默认的不带参数的构造方法。如果我们自己添加了类的构造方法（无论是否有参），Java 就不会添加默认的无参数的构造方法了。

我们一直在不知不觉地使用构造方法，这也是为什么我们在创建对象的时候后面要加一个括号（因为要调用无参的构造方法）。如果我们重载了有参的构造方法，记得都要把无参的构造方法也写出来（无论是否用到），因为这可以帮助我们在创建对象的时候少踩坑。

### 35. 构造方法有哪些特点？是否可被 override?

构造方法具有以下特点：

- **名称与类名相同**：构造方法的名称必须与类名完全一致。
- **没有返回值**：构造方法没有返回类型，且不能使用 `void` 声明。
- **自动执行**：在生成类的对象时，构造方法会自动执行，无需显式调用。

构造方法**不能被重写（override）**，但**可以被重载（overload）**。因此，一个类中可以有多个构造方法，这些构造方法可以具有不同的参数列表，以提供不同的对象初始化方式。

### 36. ⭐️面向对象三大特征

**封装：**

封装是指把一个对象的状态信息（也就是属性）隐藏在对象内部，不允许外部对象直接访问对象的内部信息。但是可以提供一些可以被外界访问的方法来操作属性。就好像我们看不到挂在墙上的空调的内部的零件信息（也就是属性），但是可以通过遥控器（方法）来控制空调。如果属性不想被外界访问，我们大可不必提供方法给外界访问。但是如果一个类没有提供给外界访问的方法，那么这个类也没有什么意义了。就好像如果没有空调遥控器，那么我们就无法操控空凋制冷，空调本身就没有意义了（当然现在还有很多其他方法 ，这里只是为了举例子）。

**继承：**

不同类型的对象，相互之间经常有一定数量的共同点。例如，小明同学、小红同学、小李同学，都共享学生的特性（班级、学号等）。同时，每一个对象还定义了额外的特性使得他们与众不同。例如小明的数学比较好，小红的性格惹人喜爱；小李的力气比较大。继承是使用已存在的类的定义作为基础建立新类的技术，新类的定义可以增加新的数据或新的功能，也可以用父类的功能，但不能选择性地继承父类。通过使用继承，可以快速地创建新的类，可以提高代码的重用，程序的可维护性，节省大量创建新类的时间 ，提高我们的开发效率。

**关于继承如下 3 点请记住：**

1. 子类拥有父类对象所有的属性和方法（包括私有属性和私有方法），但是父类中的私有属性和方法子类是无法访问，**只是拥有**。
2. 子类可以拥有自己属性和方法，即子类可以对父类进行扩展。
3. 子类可以用自己的方式实现父类的方法。（以后介绍）。

**多态：**

多态，顾名思义，表示一个对象具有多种的状态，具体表现为父类的引用指向子类的实例。

**多态的特点:**

- 对象类型和引用类型之间具有继承（类）/实现（接口）的关系；
- 引用类型变量发出的方法调用的到底是哪个类中的方法，必须在程序运行期间才能确定；
- 多态不能调用“只在子类存在但在父类不存在”的方法；
- 如果子类重写了父类的方法，真正执行的是子类重写的方法，如果子类没有重写父类的方法，执行的是父类的方法。

### 37. ⭐️接口和抽象类有什么共同点和区别？

**接口和抽象类的共同点：**

- **实例化**：接口和抽象类都不能直接实例化，只能被实现（接口）或继承（抽象类）后才能创建具体的对象。
- **抽象方法**：接口和抽象类都可以包含抽象方法。抽象方法没有方法体，必须在子类或实现类中实现。

**接口和抽象类的区别：**

- **设计目的**：接口主要用于对类的行为进行约束，你实现了某个接口就具有了对应的行为。抽象类主要用于代码复用，强调的是所属关系。
- **继承和实现**：一个类只能继承一个类（包括抽象类），因为 Java 不支持多继承。但一个类可以实现多个接口，一个接口也可以继承多个其他接口。
- **成员变量**：接口中的成员变量只能是 `public static final` 类型的，不能被修改且必须有初始值。抽象类的成员变量可以有任何修饰符（`private`, `protected`, `public`），可以在子类中被重新定义或赋值。
- **方法**：

- Java 8 之前，接口中的方法默认是 `public abstract` ，也就是只能有方法声明。自 Java 8 起，可以在接口中定义 `default`（默认） 方法和 `static` （静态）方法。 自 Java 9 起，接口可以包含 `private` 方法。
- 抽象类可以包含抽象方法和非抽象方法。抽象方法没有方法体，必须在子类中实现。非抽象方法有具体实现，可以直接在抽象类中使用或在子类中重写。

在 Java 8 及以上版本中，接口引入了新的方法类型：`default` 方法、`static` 方法和 `private` 方法。这些方法让接口的使用更加灵活。

Java 8 引入的`default` 方法用于提供接口方法的默认实现，可以在实现类中被覆盖。这样就可以在不修改实现类的情况下向现有接口添加新功能，从而增强接口的扩展性和向后兼容性。

``` Java
public interface MyInterface {
    default void defaultMethod() {
        System.out.println("This is a default method.");
    }
}
```

Java 8 引入的`static` 方法无法在实现类中被覆盖，只能通过接口名直接调用（ `MyInterface.staticMethod()`），类似于类中的静态方法。`static` 方法通常用于定义一些通用的、与接口相关的工具方法，一般很少用。

```
public interface MyInterface {
    static void staticMethod() {
        System.out.println("This is a static method in the interface.");
    }
}
```

Java 9 允许在接口中使用 `private` 方法。`private`方法可以用于在接口内部共享代码，不对外暴露。

```
public interface MyInterface {
    // default 方法
    default void defaultMethod() {
        commonMethod();
    }

    // static 方法
    static void staticMethod() {
        commonMethod();
    }

    // 私有静态方法，可以被 static 和 default 方法调用
    private static void commonMethod() {
        System.out.println("This is a private method used internally.");
    }

    // 实例私有方法，只能被 default 方法调用。
    private void instanceCommonMethod() {
        System.out.println("This is a private instance method used internally.");
    }
}
```

### 38. 深拷贝和浅拷贝区别了解吗？什么是引用拷贝？

关于深拷贝和浅拷贝区别，我这里先给结论：

- **浅拷贝**：浅拷贝会在堆上创建一个新的对象（区别于引用拷贝的一点），不过，如果原对象内部的属性是引用类型的话，浅拷贝会直接复制内部对象的引用地址，也就是说拷贝对象和原对象共用同一个内部对象。

- **深拷贝**：深拷贝会完全复制整个对象，包括这个对象所包含的内部对象。

**浅拷贝：**

浅拷贝的示例代码如下，我们这里实现了 `Cloneable` 接口，并重写了 `clone()` 方法。

`clone()` 方法的实现很简单，直接调用的是父类 `Object` 的 `clone()` 方法。

``` Java
public class Address implements Cloneable{
    private String name;
    // 省略构造函数、Getter&Setter方法
    @Override
    public Address clone() {
        try {
            return (Address) super.clone();
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }
}

public class Person implements Cloneable {
    private Address address;
    // 省略构造函数、Getter&Setter方法
    @Override
    public Person clone() {
        try {
            Person person = (Person) super.clone();
            return person;
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }
}
```

测试：

``` Java
Person person1 = new Person(new Address("武汉"));
Person person1Copy = person1.clone();
// true
System.out.println(person1.getAddress() == person1Copy.getAddress());
```

从输出结构就可以看出， `person1` 的克隆对象和 `person1` 使用的仍然是同一个 `Address` 对象。

**深拷贝：**

这里我们简单对 `Person` 类的 `clone()` 方法进行修改，连带着要把 `Person` 对象内部的 `Address` 对象一起复制。

``` Java
@Override
public Person clone() {
    try {
        Person person = (Person) super.clone();
        person.setAddress(person.getAddress().clone());
        return person;
    } catch (CloneNotSupportedException e) {
        throw new AssertionError();
    }
}
```

测试：

``` Java
Person person1 = new Person(new Address("武汉"));
Person person1Copy = person1.clone();
// false
System.out.println(person1.getAddress() == person1Copy.getAddress());
```

从输出结构就可以看出，显然 `person1` 的克隆对象和 `person1` 包含的 `Address` 对象已经是不同的了。

**那什么是引用拷贝呢？** 简单来说，引用拷贝就是两个不同的引用指向同一个对象。

我专门画了一张图来描述浅拷贝、深拷贝、引用拷贝：

![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1766238302132-8bc66050-1b07-4ab2-80bd-ab8ebf51585a.png)

### 39. Object 类的常见方法有哪些？

Object 类是一个特殊的类，是所有类的父类，主要提供了以下 11 个方法：

``` Java
/**
 * native 方法，用于返回当前运行时对象的 Class 对象，使用了 final 关键字修饰，故不允许子类重写。
 */
public final native Class<?> getClass()
/**
 * native 方法，用于返回对象的哈希码，主要使用在哈希表中，比如 JDK 中的HashMap。
 */
public native int hashCode()
/**
 * 用于比较 2 个对象的内存地址是否相等，String 类对该方法进行了重写以用于比较字符串的值是否相等。
 */
public boolean equals(Object obj)
/**
 * native 方法，用于创建并返回当前对象的一份拷贝。
 */
protected native Object clone() throws CloneNotSupportedException
/**
 * 返回类的名字实例的哈希码的 16 进制的字符串。建议 Object 所有的子类都重写这个方法。
 */
public String toString()
/**
 * native 方法，并且不能重写。唤醒一个在此对象监视器上等待的线程(监视器相当于就是锁的概念)。如果有多个线程在等待只会任意唤醒一个。
 */
public final native void notify()
/**
 * native 方法，并且不能重写。跟 notify 一样，唯一的区别就是会唤醒在此对象监视器上等待的所有线程，而不是一个线程。
 */
public final native void notifyAll()
/**
 * native方法，并且不能重写。暂停线程的执行。注意：sleep 方法没有释放锁，而 wait 方法释放了锁 ，timeout 是等待时间。
 */
public final native void wait(long timeout) throws InterruptedException
/**
 * 多了 nanos 参数，这个参数表示额外时间（以纳秒为单位，范围是 0-999999）。 所以超时的时间还需要加上 nanos 纳秒。。
 */
public final void wait(long timeout, int nanos) throws InterruptedException
/**
 * 跟之前的2个wait方法一样，只不过该方法一直等待，没有超时时间这个概念
 */
public final void wait() throws InterruptedException
/**
 * 实例被垃圾回收器回收的时候触发的操作
 */
protected void finalize() throws Throwable { }
```

### 40. == 和 equals() 的区别

`==` 对于基本类型和引用类型的作用效果是不同的：

- 对于基本数据类型来说，`==` 比较的是值。
- 对于引用数据类型来说，`==` 比较的是对象的内存地址。

因为 Java 只有值传递，所以，对于 == 来说，不管是比较基本数据类型，还是引用数据类型的变量，其本质比较的都是值，只是引用类型变量存的值是对象的地址。

`equals()` 不能用于判断基本数据类型的变量，只能用来判断两个对象是否相等。`equals()`方法存在于`Object`类中，而`Object`类是所有类的直接或间接父类，因此所有的类都有`equals()`方法。

`Object` 类 `equals()` 方法：

``` Java
public boolean equals(Object obj) {
    return (this == obj);
}
```

`equals()` 方法存在两种使用情况：

- **类没有重写** `equals()`**方法**：通过`equals()`比较该类的两个对象时，等价于通过“==”比较这两个对象，使用的默认是 `Object`类`equals()`方法。
- **类重写了** `equals()`**方法**：一般我们都重写 `equals()`方法来比较两个对象中的属性是否相等；若它们的属性相等，则返回 true(即，认为这两个对象相等)。

### 41. hashCode() 有什么用？

`hashCode()` 的作用是获取哈希码（`int` 整数），也称为散列码。这个哈希码的作用是确定该对象在哈希表中的索引位置。

**哈希表是什么**？

哈希表（Hash Table，也叫散列表）是一种非常高效的数据结构，用于实现**键-值对（key-value）**的存储和查找。它在计算机科学中广泛应用，比如编程语言中的字典（Dictionary）、Map、对象等。

**核心原理**

哈希表的底层通常是一个**数组**，通过一个叫做`哈希函数（Hash Function）`的机制，将任意类型的键（key，比如字符串、整数）转换成数组的一个索引（下标）。

过程大致如下：

1. 你想插入一个键值对：key → value
2. 哈希函数计算：index = hash(key)（得到一个整数索引）
3. 把 value 存放到数组的 array[index] 位置

查找时同样：

- 输入 key → 计算 hash(key) → 直接去数组对应位置取值

理论上，插入、删除、查找的**平均时间复杂度都是 O(1)**（常数时间），远高于数组的 O(n) 或二叉搜索树的 O(log n)。

![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1766238302097-db05dbac-c510-4b4f-a578-1917a1010876.png)

`hashCode()` 定义在 JDK 的 `Object` 类中，这就意味着 Java 中的任何类都包含有 `hashCode()` 函数。另外需要注意的是：`Object` 的 `hashCode()` 方法是本地方法，也就是用 C 语言或 C++ 实现的。

### **散列码是什么？**

散列码（也称为哈希码、Hash Code）是哈希函数（Hash Function）对某个输入数据（通常称为键 key）计算后得到的一个固定长度的整数值（有时也可能是其他形式，但最常见是整数）。

简单来说： 散列码 = hash(key)

它在哈希表（散列表）中扮演核心角色，用于将任意类型的键快速映射到数组的一个索引位置。

### 42. 为什么要有 hashCode？

**核心作用：大幅提升查找效率**

如果没有 `hashCode`，要在集合中查找一个元素是否存在，或者插入新元素时进行去重，只能通过 `equals()` 方法与集合里的元素**逐个对比**。如果集合里有 1 万个元素，最坏情况下要比对 1 万次，时间复杂度是 $O(n)$。

而引入 `hashCode` 后，它能将对象的特征映射为一个整数（哈希码）。

- 插入或查找时，先计算对象的 `hashCode`。
    
- 通过哈希算法直接定位到它在内存中的大概位置（桶位）。
    
- 这让理想情况下的查找和插入时间复杂度直接降到了 **$O(1)$**。

### 43. 为什么重写 equals() 时必须重写 hashCode() 方法？

因为两个相等的对象的 `hashCode` 值必须是相等。也就是说如果 `equals` 方法判断两个对象是相等的，那这两个对象的 `hashCode` 值也要相等。

如果重写 `equals()` 时没有重写 `hashCode()` 方法的话就可能会导致 `equals` 方法判断是相等的两个对象，`hashCode` 值却不相等。

**思考**：重写 `equals()` 时没有重写 `hashCode()` 方法的话，使用 `HashMap` 可能会出现什么问题。

**总结**：

- `equals` 方法判断两个对象是相等的，那这两个对象的 `hashCode` 值也要相等。
- 两个对象有相同的 `hashCode` 值，他们也不一定是相等的（哈希碰撞）。

### 44. ⭐️String、StringBuffer、StringBuilder 的区别？

**可变性**

`String` 是不可变的（后面会详细分析原因）。

`StringBuilder` 与 `StringBuffer` 都继承自 `AbstractStringBuilder` 类，在 `AbstractStringBuilder` 中也是使用字符数组保存字符串，不过没有使用 `final` 和 `private` 关键字修饰，最关键的是这个 `AbstractStringBuilder` 类还提供了很多修改字符串的方法比如 `append` 方法。

``` Java
abstract class AbstractStringBuilder implements Appendable, CharSequence {
    char[] value;
    public AbstractStringBuilder append(String str) {
        if (str == null)
            return appendNull();
        int len = str.length();
        ensureCapacityInternal(count + len);
        str.getChars(0, len, value, count);
        count += len;
        return this;
    }
    //...
}
```

**线程安全性**

`String` 中的对象是不可变的，也就可以理解为常量，线程安全。`AbstractStringBuilder` 是 `StringBuilder` 与 `StringBuffer` 的公共父类，定义了一些字符串的基本操作，如 `expandCapacity`、`append`、`insert`、`indexOf` 等公共方法。`StringBuffer` 对方法加了同步锁或者对调用的方法加了同步锁，所以是线程安全的。`StringBuilder` 并没有对方法进行加同步锁，所以是非线程安全的。

![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1766238302228-fbbc24b4-f6cb-4cc4-aca0-6f1ce8482330.png)

**性能**

每次对 `String` 类型进行改变的时候，都会生成一个新的 `String` 对象，然后将指针指向新的 `String` 对象。`StringBuffer` 每次都会对 `StringBuffer` 对象本身进行操作，而不是生成新的对象并改变对象引用。相同情况下使用 `StringBuilder` 相比使用 `StringBuffer` 仅能获得 10%~15% 左右的性能提升，但却要冒多线程不安全的风险。

**对于三者使用的总结：**

- 操作少量的数据: 适用 `String`
- 单线程操作字符串缓冲区下操作大量数据: 适用 `StringBuilder`
- 多线程操作字符串缓冲区下操作大量数据: 适用 `StringBuffer`

### 45. ⭐️String 为什么是不可变的?

1. 保存字符串的数组被 `final` 修饰且为私有的，并且`String` 类没有提供/暴露修改这个字符串的方法。
2. `String` 类被 `final` 修饰导致其不能被继承，进而避免了子类破坏 `String` 不可变。

### 46. Java 9 为何要将 `String` 的底层实现由 `char[]` 改成了 `byte[]` ?

新版的 String 其实支持两个编码方案：Latin-1 和 UTF-16。如果字符串中包含的汉字没有超过 Latin-1 可表示范围内的字符，那就会使用 Latin-1 作为编码方案。Latin-1 编码方案下，`byte` 占一个字节(8 位)，`char` 占用 2 个字节（16），`byte` 相较 `char` 节省一半的内存空间。

### 47. ⭐️字符串拼接用“+” 还是 StringBuilder?

Java 语言本身并不支持运算符重载，“+”和“+=”是专门为 String 类重载过的运算符，也是 Java 中仅有的两个重载过的运算符。

```
String str1 = "he";
String str2 = "llo";
String str3 = "world";
String str4 = str1 + str2 + str3;
```

上面的代码对应的字节码如下：

![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1766238302418-4ba3de2f-6a39-4d24-b936-91de0ad5d2e9.png)

可以看出，字符串对象通过“+”的字符串拼接方式，实际上是通过 `StringBuilder` 调用 `append()` 方法实现的，拼接完成之后调用 `toString()` 得到一个 `String` 对象 。

不过，在循环内使用“+”进行字符串的拼接的话，存在比较明显的缺陷：**编译器不会创建单个** `**StringBuilder**` **以复用，会导致创建过多的** `**StringBuilder**` **对象**。

``` Java
String[] arr = {"he", "llo", "world"};
String s = "";
for (int i = 0; i < arr.length; i++) {
    s += arr[i];
}
System.out.println(s);
```

`StringBuilder` 对象是在循环内部被创建的，这意味着每循环一次就会创建一个 `StringBuilder` 对象。

![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1766238302507-dabfccda-a0e7-4032-988b-55de2c5500c6.png)

如果直接使用 `StringBuilder` 对象进行字符串拼接的话，就不会存在这个问题了。

``` Java
String[] arr = {"he", "llo", "world"};
StringBuilder s = new StringBuilder();
for (String value : arr) {
    s.append(value);
}
System.out.println(s);
```

![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1766238302452-f76d01fd-75f2-4d2e-9f4e-5ebb030b272a.png)

### 48. String#equals() 和 Object#equals() 有何区别？

`String` 中的 `equals` 方法是被重写过的，比较的是 String 字符串的值是否相等。 `Object` 的 `equals` 方法是比较的对象的内存地址。

### 49. ⭐️字符串常量池的作用了解吗？

**字符串常量池** 是 JVM 为了提升性能和减少内存消耗针对字符串（String 类）专门开辟的一块区域，主要目的是为了避免字符串的重复创建。

``` Java
// 1.在字符串常量池中查询字符串对象 "ab"，如果没有则创建"ab"并放入字符串常量池
// 2.将字符串对象 "ab" 的引用赋值给 aa
String aa = "ab";
// 直接返回字符串常量池中字符串对象 "ab"，赋值给引用 bb
String bb = "ab";
System.out.println(aa==bb); // true
```

### 50. ⭐️String s1 = new String("abc");这句话创建了几个字符串对象？

先说答案：会创建 1 或 2 个字符串对象。

1. 字符串常量池中不存在 "abc"：会创建 2 个 字符串对象。一个在字符串常量池中，由 `ldc` 指令触发创建。一个在堆中，由 `new String()` 创建，并使用常量池中的 "abc" 进行初始化。
2. 字符串常量池中已存在 "abc"：会创建 1 个 字符串对象。该对象在堆中，由 `new String()` 创建，并使用常量池中的 "abc" 进行初始化。

下面开始详细分析。

1、如果字符串常量池中不存在字符串对象 “abc”，那么它首先会在字符串常量池中创建字符串对象 "abc"，然后在堆内存中再创建其中一个字符串对象 "abc"。

示例代码（JDK 1.8）：

``` Java
String s1 = new String("abc");
```

对应的字节码：

``` 
// 在堆内存中分配一个尚未初始化的 String 对象。
// #2 是常量池中的一个符号引用，指向 java/lang/String 类。
// 在类加载的解析阶段，这个符号引用会被解析成直接引用，即指向实际的 java/lang/String 类。
0 new #2 <java/lang/String>
// 复制栈顶的 String 对象引用，为后续的构造函数调用做准备。
// 此时操作数栈中有两个相同的对象引用：一个用于传递给构造函数，另一个用于保持对新对象的引用，后续将其存储到局部变量表。
3 dup
// JVM 先检查字符串常量池中是否存在 "abc"。
// 如果常量池中已存在 "abc"，则直接返回该字符串的引用；
// 如果常量池中不存在 "abc"，则 JVM 会在常量池中创建该字符串字面量并返回它的引用。
// 这个引用被压入操作数栈，用作构造函数的参数。
4 ldc #3 <abc>
// 调用构造方法，使用从常量池中加载的 "abc" 初始化堆中的 String 对象
// 新的 String 对象将包含与常量池中的 "abc" 相同的内容，但它是一个独立的对象，存储于堆中。
6 invokespecial #4 <java/lang/String.<init> : (Ljava/lang/String;)V>
// 将堆中的 String 对象引用存储到局部变量表
9 astore_1
// 返回，结束方法
10 return
```

`ldc (load constant)` 指令的确是从常量池中加载各种类型的常量，包括字符串常量、整数常量、浮点数常量，甚至类引用等。对于字符串常量，`ldc` 指令的行为如下：

1. **从常量池加载字符串**：`ldc` 首先检查字符串常量池中是否已经有内容相同的字符串对象。
2. **复用已有字符串对象**：如果字符串常量池中已经存在内容相同的字符串对象，`ldc` 会将该对象的引用加载到操作数栈上。
3. **没有则创建新对象并加入常量池**：如果字符串常量池中没有相同内容的字符串对象，JVM 会在常量池中创建一个新的字符串对象，并将其引用加载到操作数栈中。

2、如果字符串常量池中已存在字符串对象“abc”，则只会在堆中创建 1 个字符串对象“abc”。

示例代码（JDK 1.8）：

``` java
// 字符串常量池中已存在字符串对象“abc”
String s1 = "abc";
// 下面这段代码只会在堆中创建 1 个字符串对象“abc”
String s2 = new String("abc");
```

对应的字节码：

```
0 ldc #2 <abc>
2 astore_1
3 new #3 <java/lang/String>
6 dup
7 ldc #2 <abc>
9 invokespecial #4 <java/lang/String.<init> : (Ljava/lang/String;)V>
12 astore_2
13 return
```

这里就不对上面的字节码进行详细注释了，7 这个位置的 `ldc` 命令不会在堆中创建新的字符串对象“abc”，这是因为 0 这个位置已经执行了一次 `ldc` 命令，已经在堆中创建过一次字符串对象“abc”了。7 这个位置执行 `ldc` 命令会直接返回字符串常量池中字符串对象“abc”对应的引用。

### 51. String#intern 方法有什么作用?

`String.intern()` 是一个 `native` (本地) 方法，用来处理字符串常量池中的字符串对象引用。它的工作流程可以概括为以下两种情况：

1. **常量池中已有相同内容的字符串对象**：如果字符串常量池中已经有一个与调用 `intern()` 方法的字符串内容相同的 `String` 对象，`intern()` 方法会直接返回常量池中该对象的引用。
2. **常量池中没有相同内容的字符串对象**：如果字符串常量池中还没有一个与调用 `intern()` 方法的字符串内容相同的对象，`intern()` 方法会将当前字符串对象的引用添加到字符串常量池中，并返回该引用。

总结：

- `intern()` 方法的主要作用是确保字符串引用在常量池中的唯一性。
- 当调用 `intern()` 时，如果常量池中已经存在相同内容的字符串，则返回常量池中已有对象的引用；否则，将该字符串添加到常量池并返回其引用。

示例代码（JDK 1.8） :

``` Java
// s1 指向字符串常量池中的 "Java" 对象
String s1 = "Java";
// s2 也指向字符串常量池中的 "Java" 对象，和 s1 是同一个对象
String s2 = s1.intern();
// 在堆中创建一个新的 "Java" 对象，s3 指向它
String s3 = new String("Java");
// s4 指向字符串常量池中的 "Java" 对象，和 s1 是同一个对象
String s4 = s3.intern();
// s1 和 s2 指向的是同一个常量池中的对象
System.out.println(s1 == s2); // true
// s3 指向堆中的对象，s4 指向常量池中的对象，所以不同
System.out.println(s3 == s4); // false
// s1 和 s4 都指向常量池中的同一个对象
System.out.println(s1 == s4); // true
```

### 52. String 类型的变量和常量做“+”运算时发生了什么？

先来看字符串不加 `final` 关键字拼接的情况（JDK1.8）：

``` Java
String str1 = "str";
String str2 = "ing";
String str3 = "str" + "ing";
String str4 = str1 + str2;
String str5 = "string";
System.out.println(str3 == str4);//false
System.out.println(str3 == str5);//true
System.out.println(str4 == str5);//false
```

**注意**：比较 String 字符串的值是否相等，可以使用 `equals()` 方法。 `String` 中的 `equals` 方法是被重写过的。 `Object` 的 `equals` 方法是比较的对象的内存地址，而 `String` 的 `equals` 方法比较的是字符串的值是否相等。如果你使用 `==` 比较两个字符串是否相等的话，IDEA 还是提示你使用 `equals()` 方法替换。

![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1766238302956-cef0ac35-213c-4871-a53c-fff11afb851d.png)

**对于编译期可以确定值的字符串，也就是常量字符串 ，jvm 会将其存入字符串常量池。并且，字符串常量拼接得到的字符串常量在编译阶段就已经被存放字符串常量池，这个得益于编译器的优化。**

在编译过程中，Javac 编译器（下文中统称为编译器）会进行一个叫做 **常量折叠(Constant Folding)** 的代码优化。《深入理解 Java 虚拟机》中是也有介绍到：

![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1766238302810-df8b7a81-22e4-465d-b790-186f26b5b387.png)

常量折叠会把常量表达式的值求出来作为常量嵌在最终生成的代码中，这是 Javac 编译器会对源代码做的极少量优化措施之一(代码优化几乎都在即时编译器中进行)。

对于 `String str3 = "str" + "ing";` 编译器会给你优化成 `String str3 = "string";` 。

并不是所有的常量都会进行折叠，只有编译器在程序编译期就可以确定值的常量才可以：

- 基本数据类型( `byte`、`boolean`、`short`、`char`、`int`、`float`、`long`、`double`)以及字符串常量。
- `final` 修饰的基本数据类型和字符串变量
- 字符串通过 “+”拼接得到的字符串、基本数据类型之间算数运算（加减乘除）、基本数据类型的位运算（<<、>>、>>> ）

**引用的值在程序编译期是无法确定的，编译器无法对其进行优化。**

对象引用和“+”的字符串拼接方式，实际上是通过 `StringBuilder` 调用 `append()` 方法实现的，拼接完成之后调用 `toString()` 得到一个 `String` 对象 。

```
String str4 = new StringBuilder().append(str1).append(str2).toString();
```

我们在平时写代码的时候，尽量避免多个字符串对象拼接，因为这样会重新创建对象。如果需要改变字符串的话，可以使用 `StringBuilder` 或者 `StringBuffer`。

不过，字符串使用 `final` 关键字声明之后，可以让编译器当做常量来处理。

示例代码：

```
final String str1 = "str";
final String str2 = "ing";
// 下面两个表达式其实是等价的
String c = "str" + "ing";// 常量池中的对象
String d = str1 + str2; // 常量池中的对象
System.out.println(c == d);// true
```

被 `final` 关键字修饰之后的 `String` 会被编译器当做常量来处理，编译器在程序编译期就可以确定它的值，其效果就相当于访问常量。

如果 ，编译器在运行时才能知道其确切值的话，就无法对其优化。

示例代码（`str2` 在运行时才能确定其值）：

```
final String str1 = "str";
final String str2 = getStr();
String c = "str" + "ing";// 常量池中的对象
String d = str1 + str2; // 在堆上创建的新的对象
System.out.println(c == d);// false
public static String getStr() {
    return "ing";
}
```

### 53. Exception 和 Error 有什么区别？

**Java 异常类层次结构图概览**：

![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1766317203015-498a053b-ea9a-4ab4-ac8d-d64a77b772f2.png)

在 Java 中，所有的异常都有一个共同的祖先 `java.lang` 包中的 `Throwable` 类。`Throwable` 类有两个重要的子类:

- `Exception` :程序本身可以处理的异常，可以通过 `catch` 来进行捕获。`Exception` 又可以分为 Checked Exception (受检查异常，必须处理) 和 Unchecked Exception (不受检查异常，可以不处理)。
- `Error` ：`Error` 属于程序无法处理的错误 ，我们没办法通过 `catch` 来进行捕获不建议通过`catch`捕获 。例如 Java 虚拟机运行错误（`Virtual MachineError`）、虚拟机内存不够错误(`OutOfMemoryError`)、类定义错误（`NoClassDefFoundError`）等 。这些异常发生时，Java 虚拟机（JVM）一般会选择线程终止。

### 54. ClassNotFoundException 和 NoClassDefFoundError 的区别

- `ClassNotFoundException` 是Exception，发生在使用反射等动态加载时找不到类，是可预期的，可以捕获处理。
- `NoClassDefFoundError` 是Error，是编译时存在的类，在运行时链接不到了（比如 jar 包缺失），是环境问题，导致 JVM 无法继续。

### 55. ⭐️Checked Exception 和 Unchecked Exception 有什么区别？

**Checked Exception** 即 受检查异常 ，Java 代码在编译过程中，如果受检查异常没有被 `catch`或者`throws` 关键字处理的话，就没办法通过编译。

比如下面这段 IO 操作的代码：

![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1766317203002-b64b7391-4fec-4735-98cc-6eb70a41ec28.png)

除了`RuntimeException`及其子类以外，其他的`Exception`类及其子类都属于受检查异常 。常见的受检查异常有：IO 相关的异常、`ClassNotFoundException`、`SQLException`...。

**Unchecked Exception** 即 **不受检查异常** ，Java 代码在编译过程中 ，我们即使不处理不受检查异常也可以正常通过编译。

`RuntimeException` 及其子类都统称为非受检查异常，常见的有（建议记下来，日常开发中会经常用到）：

- `NullPointerException`(空指针错误)
- `IllegalArgumentException`(参数错误比如方法入参类型错误)
- `NumberFormatException`（字符串转换为数字格式错误，`IllegalArgumentException`的子类）
- `ArrayIndexOutOfBoundsException`（数组越界错误）
- `ClassCastException`（类型转换错误）
- `ArithmeticException`（算术错误）
- `SecurityException` （安全错误比如权限不够）
- `UnsupportedOperationException`(不支持的操作错误比如重复创建同一用户)

![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1766317203113-37f46199-0574-45be-a1bc-046bbb7f2329.png)

### 56. 你更倾向于使用 Checked Exception 还是 Unchecked Exception？

默认使用 Unchecked Exception，只在必要时才用 Checked Exception。

我们可以把 Unchecked Exception（比如 `NullPointerException`）看作是代码 Bug。对待 Bug，最好的方式是让它暴露出来然后去修复代码，而不是用 `try-catch` 去掩盖它。

一般来说，只在一种情况下使用 Checked Exception：当这个异常是业务逻辑的一部分，并且调用方必须处理它时。比如说，一个余额不足异常。这不是 bug，而是一个正常的业务分支，我需要用 Checked Exception 来强制调用者去处理这种情况，比如提示用户去充值。这样就能在保证关键业务逻辑完整性的同时，让代码尽可能保持简洁。

### 57. Throwable 类常用方法有哪些？

- `String getMessage()`: 返回异常发生时的详细信息
- `String toString()`: 返回异常发生时的简要描述
- `String getLocalizedMessage()`: 返回异常对象的本地化信息。使用 `Throwable` 的子类覆盖这个方法，可以生成本地化信息。如果子类没有覆盖该方法，则该方法返回的信息与 `getMessage()`返回的结果相同
- `void printStackTrace()`: 在控制台上打印 `Throwable` 对象封装的异常信息

### 58. try-catch-finally 如何使用？

- `try`块：用于捕获异常。其后可接零个或多个 `catch` 块，如果没有 `catch` 块，则必须跟一个 `finally` 块。
- `catch`块：用于处理 try 捕获到的异常。
- `finally` 块：无论是否捕获或处理异常，`finally` 块里的语句都会被执行。当在 `try` 块或 `catch` 块中遇到 `return` 语句时，`finally` 语句块将在方法返回之前被执行。

代码示例：

``` java
try {
    System.out.println("Try to do something");
    throw new RuntimeException("RuntimeException");
} catch (Exception e) {
    System.out.println("Catch Exception -> " + e.getMessage());
} finally {
    System.out.println("Finally");
}
```

输出：

```
Try to do something
Catch Exception -> RuntimeException
Finally
```

**注意：不要在 finally 语句块中使用 return!** 当 try 语句和 finally 语句中都有 return 语句时，try 语句块中的 return 语句会被忽略。这是因为 try 语句中的 return 返回值会先被暂存在一个本地变量中，当执行到 finally 语句中的 return 之后，这个本地变量的值就变为了 finally 语句中的 return 返回值。

### 59. finally 中的代码一定会执行吗？

`finally` 块的代码也不会被执行：

1. 程序所在的线程死亡。
2. 关闭 CPU。
3. 在 finally 之前虚拟机被终止运行的话，finally 中的代码就不会被执行。

### 60. 如何使用 try-with-resources 代替try-catch-finally？

1. **适用范围（资源的定义）：** 任何实现 `java.lang.AutoCloseable`或者 `java.io.Closeable` 的对象
2. **关闭资源和 finally 块的执行顺序：** 在 `try-with-resources` 语句中，任何 catch 或 finally 块在声明的资源关闭后运行

《Effective Java》中明确指出：

面对必须要关闭的资源，我们总是应该优先使用 `try-with-resources` 而不是`try-finally`。随之产生的代码更简短，更清晰，产生的异常对我们也更有用。`try-with-resources`语句让我们更容易编写必须要关闭的资源的代码，若采用`try-finally`则几乎做不到这点。

Java 中类似于`InputStream`、`OutputStream`、`Scanner`、`PrintWriter`等的资源都需要我们调用`close()`方法来手动关闭，一般情况下我们都是通过`try-catch-finally`语句来实现这个需求，如下：

``` java
//读取文本文件的内容
Scanner scanner = null;
try {
    scanner = new Scanner(new File("D://read.txt"));
    while (scanner.hasNext()) {
        System.out.println(scanner.nextLine());
    }
} catch (FileNotFoundException e) {
    e.printStackTrace();
} finally {
    if (scanner != null) {
        scanner.close();
    }
}
```

使用 Java 7 之后的 `try-with-resources` 语句改造上面的代码:

``` Java
try (Scanner scanner = new Scanner(new File("test.txt"))) {
    while (scanner.hasNext()) {
        System.out.println(scanner.nextLine());
    }
} catch (FileNotFoundException fnfe) {
    fnfe.printStackTrace();
}
```

当然多个资源需要关闭的时候，使用 `try-with-resources` 实现起来也非常简单，如果你还是用`try-catch-finally`可能会带来很多问题。

通过使用分号分隔，可以在`try-with-resources`块中声明多个资源。

```
try (BufferedInputStream bin = new BufferedInputStream(new FileInputStream(new File("test.txt")));
     BufferedOutputStream bout = new BufferedOutputStream(new FileOutputStream(new File("out.txt")))) {
    int b;
    while ((b = bin.read()) != -1) {
        bout.write(b);
    }
}
catch (IOException e) {
    e.printStackTrace();
}
```

### 61. ⭐️异常使用有哪些需要注意的地方？

- 不要把异常定义为静态变量，因为这样会导致异常栈信息错乱。每次手动抛出异常，我们都需要手动 new 一个异常对象抛出。
- 抛出的异常信息一定要有意义。
- 建议抛出更加具体的异常，比如字符串转换为数字格式错误的时候应该抛出`NumberFormatException`而不是其父类`IllegalArgumentException`。
- 避免重复记录日志：如果在捕获异常的地方已经记录了足够的信息（包括异常类型、错误信息和堆栈跟踪等），那么在业务代码中再次抛出这个异常时，就不应该再次记录相同的错误信息。重复记录日志会使得日志文件膨胀，并且可能会掩盖问题的实际原因，使得问题更难以追踪和解决。
- ……

### 62. 什么是泛型？有什么作用？

**Java 泛型（Generics）** 是 JDK 5 中引入的一个新特性，泛型 = “类型参数化” 让类、接口、方法在定义时不写死具体类型，而是在使用时再告诉它“我要用什么类型”。

编译器可以对泛型参数进行检测，并且通过泛型参数可以指定传入的对象类型。比如 `ArrayList<Person> persons = new ArrayList<Person>()` 这行代码就指明了该 `ArrayList` 对象只能传入 `Person` 对象，如果传入其他类型的对象就会报错。

``` JAva
ArrayList<E> extends AbstractList<E>
```

并且，原生 `List` 返回类型是 `Object` ，需要手动转换类型才能使用，使用泛型后编译器自动转换。

### 63. 泛型的使用方式有哪几种？

泛型一般有三种使用方式:**泛型类**、**泛型接口**、**泛型方法**。

| 字母  | 常用含义（英文全称）    | 典型使用场景              | 示例                                               |
| --- | ------------- | ------------------- | ------------------------------------------------ |
| T   | Type（类型）      | 通用类型，最常用的占位符        | `class Box<T> { T item; }`                       |
| E   | Element（元素）   | 集合中的元素              | `List<E>`<br><br>, `Set<E>`<br><br>（Java 集合框架标准） |
| K   | Key（键）        | Map 中的键             | `Map<K, V>`                                      |
| V   | Value（值）      | Map 中的值，或者泛型中的第二个类型 | `Map<K, V>`<br><br>, `Entry<K, V>`               |
| N   | Number（数字）    | 表示数字类型              | `List<N extends Number>`                         |
| S   | Second（第二个类型） | 第二个泛型参数             | `class Pair<S, T>`                               |
| U   | Third（第三个类型）  | 第三个泛型参数             | 同上                                               |
| R   | Return（返回类型）  | 常用于表示方法的返回类型        | `<R> R execute(Supplier<R> supplier)`            |

**1.泛型类**：

``` java
//此处T可以随便写为任意标识，常见的如T、E、K、V等形式的参数常用于表示泛型
//在实例化泛型类时，必须指定T的具体类型
public class Generic<T>{
    
    private T key;
    
    public Generic(T key) {
        this.key = key;
    }
    
    public T getKey(){
        return key;
    }
}
```

如何实例化泛型类：

``` java
Generic<Integer> genericInteger = new Generic<Integer>(123456);
```

**2.泛型接口**：

``` java
public interface Generator<T> {
    public T method();
}
```

实现泛型接口，不指定类型：

``` java
class GeneratorImpl<T> implements Generator<T>{
    @Override
    public T method() {
        return null;
    }
}
```

实现泛型接口，指定类型：

``` java
class GeneratorImpl implements Generator<String> {
    @Override
    public String method() {
        return "hello";
    }
}
```

**3.泛型方法**：

``` java
public static < E > void printArray( E [] inputArray ){
    for ( E element : inputArray ){
        System.out.printf( "%s ", element );
    }
    System.out.println();
}
```

使用：

``` java
// 创建不同类型数组：Integer, Double 和 Character
Integer[] intArray = { 1, 2, 3 };
String[] stringArray = { "Hello", "World" };
printArray( intArray );
printArray( stringArray );
```

注意: `public static < E > void printArray( E[] inputArray )` 一般被称为静态泛型方法;在 java 中泛型只是一个占位符，必须在传递类型后才能使用。类在实例化时才能真正的传递类型参数，由于静态方法的加载先于类的实例化，也就是说类中的泛型还没有传递真正的类型参数，静态的方法的加载就已经完成了，所以静态泛型方法是没有办法使用类上声明的泛型的。只能使用自己声明的 `<E>`。

### 64. 项目中哪里用到了泛型？

- 自定义接口通用返回结果 `CommonResult<T>` 通过参数 `T` 可根据具体的返回类型动态指定结果的数据类型
- 定义 `Excel` 处理类 `ExcelUtil<T>` 用于动态指定 `Excel` 导出的数据类型
- 构建集合工具类（参考 `Collections` 中的 `sort`, `binarySearch` 方法）。

### 65. 什么是反射？

关于反射的详细解读，请看这篇文章 [Java 反射机制详解](https://javaguide.cn/java/basis/reflection.html) 。

简单来说，Java 反射 (Reflection) 是一种**在程序运行时，动态地获取类的信息并操作类或对象（方法、属性）的能力**。

通常情况下，我们写的代码在编译时类型就已经确定了，要调用哪个方法、访问哪个字段都是明确的。但反射允许我们在**运行时**才去探知一个类有哪些方法、哪些属性、它的构造函数是怎样的，甚至可以动态地创建对象、调用方法或修改属性，哪怕这些方法或属性是私有的。

正是这种在运行时“反观自身”并进行操作的能力，使得反射成为许多**通用框架和库的基石**。它让代码更加灵活，能够处理在编译时未知的类型。

### 66. 反射有什么优缺点？

**优点：**

1. **灵活性和动态性**：反射允许程序在运行时动态地加载类、创建对象、调用方法和访问字段。这样可以根据实际需求（如配置文件、用户输入、注解等）动态地适应和扩展程序的行为，显著提高了系统的灵活性和适应性。
2. **框架开发的基础**：许多现代 Java 框架（如 Spring、Hibernate、MyBatis）都大量使用反射来实现依赖注入（DI）、面向切面编程（AOP）、对象关系映射（ORM）、注解处理等核心功能。反射是实现这些“魔法”功能不可或缺的基础工具。
3. **解耦合和通用性**：通过反射，可以编写更通用、可重用和高度解耦的代码，降低模块之间的依赖。例如，可以通过反射实现通用的对象拷贝、序列化、Bean 工具等。

**缺点：**

1. **性能开销**：反射操作通常比直接代码调用要慢。因为涉及到动态类型解析、方法查找以及 JIT 编译器的优化受限等因素。不过，对于大多数框架场景，这种性能损耗通常是可以接受的，或者框架本身会做一些缓存优化。
2. **安全性问题**：反射可以绕过 Java 语言的访问控制机制（如访问 `private` 字段和方法），破坏了封装性，可能导致数据泄露或程序被恶意篡改。此外，还可以绕过泛型检查，带来类型安全隐患。
3. **代码可读性和维护性**：过度使用反射会使代码变得复杂、难以理解和调试。错误通常在运行时才会暴露，不像编译期错误那样容易发现。

相关阅读：[Java Reflection: Why is it so slow?](https://stackoverflow.com/questions/1392351/java-reflection-why-is-it-so-slow) 。

### 67. 反射的应用场景？

我们平时写业务代码可能很少直接跟 Java 的反射（Reflection）打交道。但你可能没意识到，你天天都在享受反射带来的便利！**很多流行的框架，比如 Spring/Spring Boot、MyBatis 等，底层都大量运用了反射机制**，这才让它们能够那么灵活和强大。

下面简单列举几个最场景的场景帮助大家理解。

**1.依赖注入与控制反转（IoC）**

以 Spring/Spring Boot 为代表的 IoC 框架，会在启动时扫描带有特定注解（如 `@Component`, `@Service`, `@Repository`, `@Controller`）的类，利用反射实例化对象（Bean），并通过反射注入依赖（如 `@Autowired`、构造器注入等）。

**2.注解处理**

注解本身只是个“标记”，得有人去读这个标记才知道要做什么。反射就是那个“读取器”。框架通过反射检查类、方法、字段上有没有特定的注解，然后根据注解信息执行相应的逻辑。比如，看到 `@Value`，就用反射读取注解内容，去配置文件找对应的值，再用反射把值设置给字段。

**3.动态代理与 AOP**

想在调用某个方法前后自动加点料（比如打日志、开事务、做权限检查）？AOP（面向切面编程）就是干这个的，而动态代理是实现 AOP 的常用手段。JDK 自带的动态代理（Proxy 和 InvocationHandler）就离不开反射。代理对象在内部调用真实对象的方法时，就是通过反射的 `Method.invoke` 来完成的。

``` Java
public class DebugInvocationHandler implements InvocationHandler {
    private final Object target; // 真实对象

    public DebugInvocationHandler(Object target) { this.target = target; }

    // proxy: 代理对象, method: 被调用的方法, args: 方法参数
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("切面逻辑：调用方法 " + method.getName() + " 之前");
        // 通过反射调用真实对象的同名方法
        Object result = method.invoke(target, args);
        System.out.println("切面逻辑：调用方法 " + method.getName() + " 之后");
        return result;
    }
}
```

**4.对象关系映射（ORM）**

像 MyBatis、Hibernate 这种框架，能帮你把数据库查出来的一行行数据，自动变成一个个 Java 对象。它是怎么知道数据库字段对应哪个 Java 属性的？还是靠反射。它通过反射获取 Java 类的属性列表，然后把查询结果按名字或配置对应起来，再用反射调用 setter 或直接修改字段值。反过来，保存对象到数据库时，也是用反射读取属性值来拼 SQL。

### 68. 如何实现动态代理？

关于 Java 代理的详细介绍，可以看看笔者写的 [Java 代理模式详解](https://javaguide.cn/java/basis/proxy.html)这篇文章。

动态代理是一种非常强大的设计模式，它允许我们在**不修改源代码**的情况下，对一个类或对象的方法进行**功能增强（Enhancement）**。

在 Java 中，实现动态代理最主流的方式有两种：**JDK 动态代理** 和 **CGLIB 动态代理**。

**第一种：JDK 动态代理**

Java 官方提供的，其核心要求是目标类必须实现一个或多个接口。JDK 动态代理在运行时，会利用 `Proxy.newProxyInstance()` 方法，动态地创建一个实现了这些接口的代理类的实例。这个代理类在内存中生成，你看不到它的 `.java` 或 `.class` 文件。

当你调用代理对象的任何一个方法时，这个调用都会被转发到我们提供的一个 `InvocationHandler` 接口的 `invoke` 方法中。在 `invoke` 方法里，我们就可以在调用原始方法（目标方法）之前或之后，加入我们自己的增强逻辑。

**第二种：CGLIB 动态代理**

CGLIB 是一个第三方的代码生成库。它的原理与 JDK 完全不同，它不要求被代理的类实现接口。它在运行时，动态生成目标类的子类作为代理类（通过 ASM 字节码操作技术）。然后，它会重写父类（也就是被代理类）中所有非 `final`、`private` 和 `static` 的方法。

当你调用代理对象的任何一个方法时，这个调用会被 CGLIB 的 `MethodInterceptor` 接口的 `intercept` 方法拦截。和 `InvocationHandler` 的 `invoke` 方法一样，我们可以在 `intercept` 方法里，在调用原始的父类方法之前或之后，加入我们的增强逻辑。

### 69. 静态代理和动态代理有什么区别？

静态代理和动态代理的核心差异在于 **代理关系的确定时机、实现灵活性及维护成本** 。

| 对比维度     | 静态代理 (Static Proxy)                          | 动态代理 (Dynamic Proxy)                             |
| -------- | -------------------------------------------- | ------------------------------------------------ |
| 代理关系确定时机 | 编译期（编译后生成固定的 `.class`字节码文件）                  | 运行时（动态生成代理类字节码并加载到 JVM）                          |
| 实现方式     | 手动编写代理类，需与目标类实现同一接口，一对一绑定                    | 无需手动编写代理类，通过 `Handler`/`Interceptor`封装增强逻辑，一对多复用 |
| 接口依赖     | 必须实现接口（代理类与目标类遵循同一接口规范）                      | 支持代理接口或直接代理实现类                                   |
| 代码量与维护性  | 代码量大（目标类越多，代理类越多），维护成本高；接口新增方法时，目标类与代理类需同步修改 | 代码量极少（通用增强逻辑可复用），维护性好；与接口解耦，接口变更不影响代理逻辑          |
| 核心优势     | 实现简单、逻辑直观，无额外框架依赖                            | 灵活性强、复用性高，降低重复编码，适配复杂场景                          |
| 典型应用场景   | 简单的装饰器模式、少量固定类的增强需求                          | Spring AOP、RPC 框架（如 Dubbo）、ORM 框架                |

### 70. ⭐️JDK 动态代理和 CGLIB 动态代理有什么区别？

1. JDK 动态代理是官方的，它要求被代理的类必须实现接口。它的原理是动态生成一个接口的实现类来作为代理。CGLIB 是第三方的，它不需要接口。它的原理是动态生成一个被代理类的子类来作为代理。但也正因为是继承，所以它不能代理 `final` 的类，被代理的方法也不能是 `final` 或 `private` 。
2. 就二者的效率来说，大部分情况都是 JDK 动态代理更优秀，随着 JDK 版本的升级，这个优势更加明显。

### 71. ⭐️介绍一下动态代理在框架中的实际应用场景

动态代理最典型的应用场景就是**Spring AOP**。

AOP(Aspect-Oriented Programming:面向切面编程)能够将那些与业务无关，却为业务模块所共同调用的逻辑或责任（例如事务处理、日志管理、权限控制等）封装起来，便于减少系统的重复代码，降低模块间的耦合度，并有利于未来的可拓展性和可维护性。

Spring AOP 就是基于动态代理的，如果要代理的对象，实现了某个接口，那么 Spring AOP 会使用 **JDK Proxy**，去创建代理对象，而对于没有实现接口的对象，就无法使用 JDK Proxy 去进行代理了，这时候 Spring AOP 会使用 **Cglib** 生成一个被代理对象的子类来作为代理，如下图所示：

![](https://cdn.nlark.com/yuque/0/2025/jpeg/58546395/1766317203028-959ce476-e1dd-4bc6-b5c8-be78f9187c8d.jpg.jpeg)

### 72. 何谓注解？

`Annotation` （注解） 是 Java5 开始引入的新特性，可以看作是一种特殊的注释，主要用于修饰类、方法或者变量，提供某些信息供程序在编译或者运行时使用。

注解本质是一个继承了`Annotation` 的特殊接口：

``` Java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.SOURCE)
public @interface Override {

}

public interface Override extends Annotation{

}
```

JDK 提供了很多内置的注解（比如 `@Override`、`@Deprecated`），同时，我们还可以自定义注解。

### 73. 注解的解析方法有哪几种？

注解只有被解析之后才会生效，常见的解析方法有两种：

- **编译期直接扫描**：编译器在编译 Java 代码的时候扫描对应的注解并处理，比如某个方法使用`@Override` 注解，编译器在编译的时候就会检测当前的方法是否重写了父类对应的方法。
- **运行期通过反射处理**：像框架中自带的注解(比如 Spring 框架的 `@Value`、`@Component`)都是通过反射来进行处理的。

### 74. 何谓 SPI?

关于 SPI 的详细解读，请看这篇文章 [Java SPI 机制详解](https://javaguide.cn/java/basis/spi.html) 。

SPI 即 Service Provider Interface ，字面意思就是：“服务提供者的接口”，我的理解是：专门提供给服务提供者或者扩展框架功能的开发者去使用的一个接口。

SPI 将服务接口和具体的服务实现分离开来，将服务调用方和服务实现者解耦，能够提升程序的扩展性、可维护性。修改或者替换服务实现并不需要修改调用方。

很多框架都使用了 Java 的 SPI 机制，比如：Spring 框架、数据库加载驱动、日志接口、以及 Dubbo 的扩展实现等等。

![](https://cdn.nlark.com/yuque/0/2025/jpeg/58546395/1766317203452-b3c82cc5-44bf-4e6f-8f48-c1c00813a856.jpg.jpeg)

### 75. SPI 和 API 有什么区别？

| 对比项   | API                               | SPI                                             |
| ----- | --------------------------------- | ----------------------------------------------- |
| 全称    | Application Programming Interface | Service Provider Interface                      |
| 谁调用谁  | 调用方 → 被调用方（你用别人的功能）               | 框架/组件 → 实现方（别人实现你的接口）                           |
| 谁提供接口 | 被调用的服务/库提供                        | 框架/组件提供                                         |
| 谁写实现  | 调用方不用写实现                          | 第三方/插件开发者写实现                                    |
| 典型目的  | 拿来直接用                             | 做扩展点、插件化、解耦                                     |
| 代表性例子 | HttpClient、MyBatis的SqlSession     | JDBC Driver、Dubbo Extension、Spring Boot Starter |


### 76. SPI 的优缺点？

通过 SPI 机制能够大大地提高接口设计的灵活性，但是 SPI 机制也存在一些缺点，比如：

- 需要遍历加载所有的实现类，不能做到按需加载，这样效率还是相对较低的。
- 当多个 `ServiceLoader` 同时 `load` 时，会有并发问题。

### 什么是序列化?什么是反序列化?

关于序列化和反序列化的详细解读，请看这篇文章 [Java 序列化详解](https://javaguide.cn/java/basis/serialization.html) ，里面涉及到的知识点和面试题更全面。

如果我们需要持久化 Java 对象比如将 Java 对象保存在文件中，或者在网络传输 Java 对象，这些场景都需要用到序列化。

简单来说：

- **序列化**：将数据结构或对象转换成可以存储或传输的形式，通常是二进制字节流，也可以是 JSON, XML 等文本格式
- **反序列化**：将在序列化过程中所生成的数据转换为原始数据结构或者对象的过程

对于 Java 这种面向对象编程语言来说，我们序列化的都是对象（Object）也就是实例化后的类(Class)，但是在 C++这种半面向对象的语言中，struct(结构体)定义的是数据结构类型，而 class 对应的是对象类型。

下面是序列化和反序列化常见应用场景：

- 对象在进行网络传输（比如远程方法调用 RPC 的时候）之前需要先被序列化，接收到序列化的对象之后需要再进行反序列化；
- 将对象存储到文件之前需要进行序列化，将对象从文件中读取出来需要进行反序列化；
- 将对象存储到数据库（如 Redis）之前需要用到序列化，将对象从缓存数据库中读取出来需要反序列化；
- 将对象存储到内存之前需要进行序列化，从内存中读取出来之后需要进行反序列化。

维基百科是如是介绍序列化的：

**序列化**（serialization）在计算机科学的数据处理中，是指将数据结构或对象状态转换成可取用格式（例如存成文件，存于缓冲，或经由网络中发送），以留待后续在相同或另一台计算机环境中，能恢复原先状态的过程。依照序列化格式重新获取字节的结果时，可以利用它来产生与原始对象相同语义的副本。对于许多对象，像是使用大量引用的复杂对象，这种序列化重建的过程并不容易。面向对象中的对象序列化，并不概括之前原始对象所关系的函数。这种过程也称为对象编组（marshalling）。从一系列字节提取数据结构的反向操作，是反序列化（也称为解编组、deserialization、unmarshalling）。

综上：**序列化的主要目的是通过网络传输对象或者说是将对象存储到文件系统、数据库、内存中。**

![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1766317203400-c51b46b1-7813-41b9-b98c-f5093c688db1.png)


**序列化协议对应于 TCP/IP 4 层模型的哪一层？**

我们知道网络通信的双方必须要采用和遵守相同的协议。TCP/IP 四层模型是下面这样的，序列化协议属于哪一层呢？

1. 应用层
2. 传输层
3. 网络层
4. 网络接口层

![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1766317203424-67e4bc57-79ee-42a0-b25f-1a37899b1550.png)

如上图所示，OSI 七层协议模型中，表示层做的事情主要就是对应用层的用户数据进行处理转换为二进制流。反过来的话，就是将二进制流转换成应用层的用户数据。这不就对应的是序列化和反序列化么？

因为，OSI 七层协议模型中的应用层、表示层和会话层对应的都是 TCP/IP 四层模型中的应用层，所以序列化协议属于 TCP/IP 协议应用层的一部分。

### 77. 如果有些字段不想进行序列化怎么办？

对于不想进行序列化的变量，使用 `transient` 关键字修饰。

`transient` 关键字的作用是：阻止实例中那些用此关键字修饰的变量序列化；当对象被反序列化时，被 `transient` 修饰的变量值不会被持久化和恢复。

关于 `transient` 还有几点注意：

- `transient` 只能修饰变量，不能修饰类和方法。
- `transient` 修饰的变量，在反序列化后变量值将会被置成类型的默认值。例如，如果是修饰 `int` 类型，那么反序列后结果就是 `0`。
- `static` 变量因为不属于任何对象(Object)，所以无论有没有 `transient` 关键字修饰，均不会被序列化。

### 78. 常见序列化协议有哪些？

JDK 自带的序列化方式一般不会用 ，因为序列化效率低并且存在安全问题。比较常用的序列化协议有 Hessian、Kryo、Protobuf、ProtoStuff，这些都是基于二进制的序列化协议。

像 JSON 和 XML 这种属于文本类序列化方式。虽然可读性比较好，但是性能较差，一般不会选择。

### 79. 为什么不推荐使用 JDK 自带的序列化？

我们很少或者说几乎不会直接使用 JDK 自带的序列化方式，主要原因有下面这些原因：

- **不支持跨语言调用** : 如果调用的是其他语言开发的服务的时候就不支持了。
- **性能差**：相比于其他序列化框架性能更低，主要原因是序列化之后的字节数组体积较大，导致传输成本加大。
- **存在安全问题**：序列化和反序列化本身并不存在问题。但当输入的反序列化的数据可被用户控制，那么攻击者即可通过构造恶意输入，让反序列化产生非预期的对象，在此过程中执行构造的任意代码。

相关阅读：[应用安全：JAVA 反序列化漏洞之殇](https://cryin.github.io/blog/secure-development-java-deserialization-vulnerability/) 。

### 80. Java IO 流了解吗？

IO 即 `Input/Output`，输入和输出。数据输入到计算机内存的过程即输入，反之输出到外部存储（比如数据库，文件，远程主机）的过程即输出。数据传输过程类似于水流，因此称为 IO 流。IO 流在 Java 中分为输入流和输出流，而根据数据的处理方式又分为字节流和字符流。

Java IO 流的 40 多个类都是从如下 4 个抽象类基类中派生出来的。

- `InputStream`/`Reader`: 所有的输入流的基类，前者是字节输入流，后者是字符输入流。
- `OutputStream`/`Writer`: 所有输出流的基类，前者是字节输出流，后者是字符输出流。

### 81. I/O 流为什么要分为字节流和字符流呢?

- **字节流**：处理**原始二进制数据**（图片、视频、exe、zip），按8位字节读写，**无字符概念**。
- **字符流**：处理**人类可读文本**（txt、json、html），按16位Unicode字符读写，**自动处理多国语言编码**。

### 82. 什么是语法糖？

**语法糖（Syntactic sugar）** 代指的是编程语言为了方便程序员开发程序而设计的一种特殊语法，这种语法对编程语言的功能并没有影响。实现相同的功能，基于语法糖写出来的代码往往更简单简洁且更易阅读。

举个例子，Java 中的 `for-each` 就是一个常用的语法糖，其原理其实就是基于普通的 for 循环和迭代器。

```
String[] strs = {"JavaGuide", "公众号：JavaGuide", "博客：https://javaguide.cn/"};
for (String s : strs) {
    System.out.println(s);
}
```

不过，JVM 其实并不能识别语法糖，Java 语法糖要想被正确执行，需要先通过编译器进行解糖，也就是在程序编译阶段将其转换成 JVM 认识的基本语法。这也侧面说明，Java 中真正支持语法糖的是 Java 编译器而不是 JVM。如果你去看`com.sun.tools.javac.main.JavaCompiler`的源码，你会发现在`compile()`中有一个步骤就是调用`desugar()`，这个方法就是负责解语法糖的实现的。

### 83. Java 中有哪些常见的语法糖？

Java 中最常用的语法糖主要有泛型、自动拆装箱、变长参数、枚举、内部类、增强 for 循环、try-with-resources 语法、lambda 表达式等。