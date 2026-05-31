## 一、SpringMVC简介
#### 1、什么是MVC
MVC是一种软件架构的思想，将软件按照模型、视图、控制器来划分

M：Model，模型层，指工程中的JavaBean，作用是处理数据

JavaBean分为两类：

+ 一类称为实体类Bean：专门存储业务数据的，如 Student、User 等
+ 一类称为业务处理 Bean：指 Service 或 Dao 对象，专门用于处理业务逻辑和数据访问。

V：View，视图层，指工程中的html或jsp等页面，作用是与用户进行交互，展示数据

C：Controller，控制层，指工程中的servlet，作用是接收请求和响应浏览器

MVC的工作流程：  
用户通过视图层发送请求到服务器，在服务器中请求被Controller接收，Controller调用相应的Model层处理请求，处理完毕将结果返回到Controller，Controller再根据请求处理的结果找到相应的View视图，渲染数据后最终响应给浏览器

#### 2、什么是SpringMVC
SpringMVC是Spring的一个后续产品，是Spring的一个子项目

SpringMVC 是 Spring 为表述层开发提供的一整套完备的解决方案。在表述层框架历经 Strust、WebWork、Strust2 等诸多产品的历代更迭之后，目前业界普遍选择了 SpringMVC 作为 Java EE 项目表述层开发的**首选方案**。

> 注：三层架构分为表述层（或表示层）、业务逻辑层、数据访问层，表述层表示前台页面和后台servlet
>

#### 3、SpringMVC的特点
+ **Spring 家族原生产品**，与 IOC 容器等基础设施无缝对接
+ **基于原生的Servlet**，通过了功能强大的**前端控制器DispatcherServlet**，对请求和响应进行统一处理
+ 表述层各细分领域需要解决的问题**全方位覆盖**，提供**全面解决方案**
+ **代码清新简洁**，大幅度提升开发效率
+ 内部组件化程度高，可插拔式组件**即插即用**，想要什么功能配置相应组件即可
+ **性能卓著**，尤其适合现代大型、超大型互联网项目要求

## 二、HelloWorld
#### 1、开发环境
IDE：idea 2019.2

构建工具：maven3.5.4

服务器：tomcat7

Spring版本：5.3.1

#### 2、创建maven工程
###### a>添加web模块
###### b>打包方式：war
###### c>引入依赖
```xml
<dependencies>
  <!-- SpringMVC -->
  <dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-webmvc</artifactId>
    <version>5.3.1</version>
  </dependency>

  <!-- 日志 -->
  <dependency>
    <groupId>ch.qos.logback</groupId>
    <artifactId>logback-classic</artifactId>
    <version>1.2.3</version>
  </dependency>

  <!-- ServletAPI -->
  <dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>javax.servlet-api</artifactId>
    <version>3.1.0</version>
    <scope>provided</scope>
  </dependency>

  <!-- Spring5和Thymeleaf整合包 -->
  <dependency>
    <groupId>org.thymeleaf</groupId>
    <artifactId>thymeleaf-spring5</artifactId>
    <version>3.0.12.RELEASE</version>
  </dependency>
</dependencies>
```

注：由于 Maven 的传递性，我们不必将所有需要的包全部配置依赖，而是配置最顶端的依赖，其他靠传递性导入。

![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1754593106484-34e08590-9f3e-4b50-a17d-eeed0565f71a.png)

#### 3、配置web.xml
注册SpringMVC的前端控制器DispatcherServlet

###### a>默认配置方式
此配置作用下，SpringMVC的配置文件默认位于WEB-INF下，默认名称为`<servlet-name>-servlet.xml`，例如，以下配置所对应SpringMVC的配置文件位于WEB-INF下，文件名为`springMVC-servlet.xml`

```  xml
<web-app>

    <!-- 
        配置 Spring MVC 的前端控制器 DispatcherServlet
        所有请求都会经过这个 Servlet 进行统一调度
    -->
    <servlet>
        <!-- Servlet 的名称，Spring 会根据此名称查找对应的配置文件 -->
        <!-- 默认查找 /WEB-INF/springMVC-servlet.xml -->
        <servlet-name>springMVC</servlet-name>

        <!-- DispatcherServlet 是 Spring MVC 的核心控制器（前端控制器） -->
        <!-- 负责接收请求、调用处理器、渲染视图等核心流程 -->
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    </servlet>

    <!-- 配置 Servlet 的映射规则 -->
    <servlet-mapping>
        <!-- 与上面 servlet-name 保持一致 -->
        <servlet-name>springMVC</servlet-name>

        <!-- 
            url-pattern 为 /，表示拦截所有请求（除 .jsp 文件外）
            包括：/user、/hello、/api/list 等
            注意：/ 与 /* 的区别：
              /  —— 不拦截 .jsp，但拦截静态资源（需额外配置放行）
              /* —— 拦截一切，包括 .jsp（会导致 JSP 无法解析，不推荐）
        -->
        <url-pattern>/</url-pattern>
    </servlet-mapping>

</web-app>
```

###### b>扩展配置方式
可通过init-param标签设置SpringMVC配置文件的位置和名称，通过load-on-startup标签设置SpringMVC前端控制器DispatcherServlet的初始化时间

``` xml
<servlet>
  <servlet-name>springMVC</servlet-name>
  <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
  <!-- 通过初始化参数指定SpringMVC配置文件的位置和名称 -->
  <init-param>
    <!-- contextConfigLocation为固定值 -->
    <param-name>contextConfigLocation</param-name>
    <!-- 使用classpath:表示从类路径查找配置文件，例如maven工程中的src/main/resources -->
    <param-value>classpath:springMVC.xml</param-value>
  </init-param>
  <!-- 
  作为框架的核心组件，在启动过程中有大量的初始化操作要做
  而这些操作放在第一次请求时才执行会严重影响访问速度
  因此需要通过此标签将启动控制DispatcherServlet的初始化时间提前到服务器启动时
  -->
  <load-on-startup>1</load-on-startup>
</servlet>
<servlet-mapping>
  <servlet-name>springMVC</servlet-name>
  <!--
  设置springMVC的核心控制器所能处理的请求的请求路径
  /所匹配的请求可以是/login或.html或.js或.css方式的请求路径
  但是/不能匹配.jsp请求路径的请求
  -->
  <url-pattern>/</url-pattern>
</servlet-mapping>
```

> 注：
>
> `<url-pattern>`标签中使用/和/*的区别：
>
> /所匹配的请求可以是/login或.html或.js或.css方式的请求路径，但是/不能匹配.jsp请求路径的请求
>
> 因此就可以避免在访问jsp页面时，该请求被DispatcherServlet处理，从而找不到相应的页面
>
> /*则能够匹配所有请求，例如在使用过滤器时，若需要对所有请求进行过滤，就需要使用/*的写法
>

#### 4、创建请求控制器
由于前端控制器对浏览器发送的请求进行了统一的处理，但是具体的请求有不同的处理过程，因此需要创建处理具体请求的类，即请求控制器

请求控制器中每一个处理请求的方法成为控制器方法

因为SpringMVC的控制器由一个POJO（普通的Java类）担任，因此需要通过@Controller注解将其标识为一个控制层组件，交给Spring的IoC容器管理，此时SpringMVC才能够识别控制器的存在

```java
@Controller
public class HelloController {

}
```

#### 5、创建springMVC的配置文件
```xml
<!-- 自动扫描包 -->
<context:component-scan base-package="com.atguigu.mvc.controller"/>

<!-- 配置Thymeleaf视图解析器 -->
<bean id="viewResolver" class="org.thymeleaf.spring5.view.ThymeleafViewResolver">
  <property name="order" value="1"/>
  <property name="characterEncoding" value="UTF-8"/>
  <property name="templateEngine">
    <bean class="org.thymeleaf.spring5.SpringTemplateEngine">
      <property name="templateResolver">
        <bean class="org.thymeleaf.spring5.templateresolver.SpringResourceTemplateResolver">

          <!-- 视图前缀 -->
          <property name="prefix" value="/WEB-INF/templates/"/>

          <!-- 视图后缀 -->
          <property name="suffix" value=".html"/>
          <property name="templateMode" value="HTML5"/>
          <property name="characterEncoding" value="UTF-8" />
        </bean>
      </property>
    </bean>
  </property>
</bean>

<!-- 
处理静态资源，例如html、js、css、jpg
若只设置该标签，则只能访问静态资源，其他请求则无法访问
此时必须设置<mvc:annotation-driven/>解决问题
-->
<mvc:default-servlet-handler/>

<!-- 开启mvc注解驱动 -->
<mvc:annotation-driven>
  <mvc:message-converters>
    <!-- 处理响应中文内容乱码 -->
    <bean class="org.springframework.http.converter.StringHttpMessageConverter">
      <property name="defaultCharset" value="UTF-8" />
      <property name="supportedMediaTypes">
        <list>
          <value>text/html</value>
          <value>application/json</value>
        </list>
      </property>
    </bean>
  </mvc:message-converters>
</mvc:annotation-driven>
```

#### 6、测试HelloWorld
###### a>实现对首页的访问
在请求控制器中创建处理请求的方法

```java
// @RequestMapping注解：处理请求和控制器方法之间的映射关系
// @RequestMapping注解的value属性可以通过请求地址匹配请求，/表示的当前工程的上下文路径
// localhost:8080/springMVC/
@RequestMapping("/")
public String index() {
    //设置视图名称
    return "index";
}
```

###### b>通过超链接跳转到指定页面
在主页index.html中设置超链接

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
  <head>
    <meta charset="UTF-8">
    <title>首页</title>
  </head>
  <body>
    <h1>首页</h1>
    <a th:href="@{/hello}">HelloWorld</a><br/>
  </body>
</html>
```

在请求控制器中创建处理请求的方法

``` java
@RequestMapping("/hello")
public String HelloWorld() {
    return "target";
}
```

#### 7、总结
浏览器发送请求，若请求地址符合前端控制器的url-pattern，该请求就会被前端控制器DispatcherServlet处理。前端控制器会读取SpringMVC的核心配置文件，通过扫描组件找到控制器，将请求地址和控制器中@RequestMapping注解的value属性值进行匹配，若匹配成功，该注解所标识的控制器方法就是处理请求的方法。处理请求的方法需要返回一个字符串类型的视图名称，该视图名称会被视图解析器解析，加上前缀和后缀组成视图的路径，通过Thymeleaf对视图进行渲染，最终转发到视图所对应页面

## 三、@RequestMapping注解
#### 1、@RequestMapping注解的功能
从注解名称上我们可以看到，@RequestMapping注解的作用就是将请求和处理请求的控制器方法关联起来，建立映射关系。

SpringMVC 接收到指定的请求，就会来找到在映射关系中对应的控制器方法来处理这个请求。

#### 2、@RequestMapping注解的位置

@RequestMapping标识一个类：设置映射请求的请求路径的初始信息

@RequestMapping标识一个方法：设置映射请求请求路径的具体信息

``` java
@Controller
@RequestMapping("/test")
public class RequestMappingController {
    //此时请求映射所映射的请求的请求路径为：/test/testRequestMapping
    @RequestMapping("/testRequestMapping")
    public String testRequestMapping(){
        return "success";
    }
}
```

#### 3、@RequestMapping注解的value属性
@RequestMapping注解的value属性通过请求的请求地址匹配请求映射

@RequestMapping注解的value属性是一个字符串类型的数组，表示该请求映射能够匹配多个请求地址所对应的请求

@RequestMapping注解的value属性必须设置，至少通过请求地址匹配请求映射

``` html
<a th:href="@{/testRequestMapping}">测试@RequestMapping的value属性-->/testRequestMapping</a><br>
<a th:href="@{/test}">测试@RequestMapping的value属性-->/test</a><br>
```

``` java
@RequestMapping(
    value = {"/testRequestMapping", "/test"}
)
public String testRequestMapping(){
    return "success";
}
```

#### 4、@RequestMapping注解的method属性

@RequestMapping注解的method属性通过请求的请求方式（get或post）匹配请求映射

@RequestMapping注解的method属性是一个RequestMethod类型的数组，表示该请求映射能够匹配多种请求方式的请求

若当前请求的请求地址满足请求映射的value属性，但是请求方式不满足method属性，则浏览器报错405：Request method ‘POST’ not supported

``` html
<a th:href="@{/test}">测试@RequestMapping的value属性-->/test</a><br>
<form th:action="@{/test}" method="post">
  <input type="submit">
</form>
```

```java
@RequestMapping(
    value = {"/testRequestMapping", "/test"},
    method = {RequestMethod.GET, RequestMethod.POST}
)
public String testRequestMapping(){
    return "success";
}
```

> 注：
>
> 1、对于处理指定请求方式的控制器方法，SpringMVC中提供了@RequestMapping的派生注解
>
> 处理get请求的映射–>@GetMapping
>
> 处理post请求的映射–>@PostMapping
>
> 处理put请求的映射–>@PutMapping
>
> 处理delete请求的映射–>@DeleteMapping
>
> 2、常用的请求方式有get，post，put，delete
>
> 但是目前浏览器只支持get和post，若在form表单提交时，为method设置了其他请求方式的字符串（put或delete），则按照默认的请求方式get处理
>
> 若要发送put和delete请求，则需要通过spring提供的过滤器HiddenHttpMethodFilter，在RESTful部分会讲到
>

#### 5、@RequestMapping注解的params属性（了解）
@RequestMapping注解的params属性通过请求的请求参数匹配请求映射

@RequestMapping注解的params属性是一个字符串类型的数组，可以通过四种表达式设置请求参数和请求映射的匹配关系

“param”：要求请求映射所匹配的请求必须携带param请求参数

“!param”：要求请求映射所匹配的请求必须不能携带param请求参数

“param=value”：要求请求映射所匹配的请求必须携带param请求参数且param=value

“param!=value”：要求请求映射所匹配的请求必须携带param请求参数但是param!=value

```xml
<a th:href="@{/test(username='admin',password=123456)">测试@RequestMapping的params属性-->/test</a><br>
```

``` java
@RequestMapping(
    value = {"/testRequestMapping", "/test"}
    ,method = {RequestMethod.GET, RequestMethod.POST}
    ,params = {"username","password!=123456"}
)
public String testRequestMapping(){
    return "success";
}
```

> 注：
>
> 若当前请求满足@RequestMapping注解的value和method属性，但是不满足params属性，此时页面回报错400：Parameter conditions “username, password!=123456” not met for actual request parameters: username={admin}, password={123456}
>

#### 6、@RequestMapping注解的headers属性（了解）

@RequestMapping注解的headers属性通过请求的请求头信息匹配请求映射

@RequestMapping注解的headers属性是一个字符串类型的数组，可以通过四种表达式设置请求头信息和请求映射的匹配关系

“header”：要求请求映射所匹配的请求必须携带header请求头信息

“!header”：要求请求映射所匹配的请求必须不能携带header请求头信息

“header=value”：要求请求映射所匹配的请求必须携带header请求头信息且header=value

“header!=value”：要求请求映射所匹配的请求必须携带header请求头信息且header!=value

若当前请求满足@RequestMapping注解的value和method属性，但是不满足headers属性，此时页面显示404错误，即资源未找到

#### 7、SpringMVC支持ant风格的路径
？：表示任意的单个字符

*：表示任意的0个或多个字符

**：表示任意的一层或多层目录

注意：在使用**时，只能使用/**/xxx的方式

#### 8、SpringMVC支持路径中的占位符（重点）

原始方式：/deleteUser?id=1

rest方式：/deleteUser/1

SpringMVC路径中的占位符常用于RESTful风格中，当请求路径中将某些数据通过路径的方式传输到服务器中，就可以在相应的@RequestMapping注解的value属性中通过占位符{xxx}表示传输的数据，在通过@PathVariable注解，将占位符所表示的数据赋值给控制器方法的形参

```xml
<a th:href="@{/testRest/1/admin}">测试路径中的占位符-->/testRest</a><br>
```

``` java
@RequestMapping("/testRest/{id}/{username}")
public String testRest(@PathVariable("id") String id, @PathVariable("username") String username){
    System.out.println("id:"+id+",username:"+username);
    return "success";
}
//最终输出的内容为-->id:1,username:admin
```

## 四、SpringMVC获取请求参数
#### 1、通过ServletAPI获取
将HttpServletRequest作为控制器方法的形参，此时HttpServletRequest类型的参数表示封装了当前请求的请求报文的对象

``` java
@RequestMapping("/testParam")
public String testParam(HttpServletRequest request){
    String username = request.getParameter("username");
    String password = request.getParameter("password");
    System.out.println("username:"+username+",password:"+password);
    return "success";
}
```

#### 2、通过控制器方法的形参获取请求参数
在控制器方法的形参位置，设置和请求参数同名的形参，当浏览器发送请求，匹配到请求映射时，在DispatcherServlet中就会将请求参数赋值给相应的形参

``` html
<a th:href="@{/testParam(username='admin',password=123456)}">测试获取请求参数-->/testParam</a><br>
```

``` java
@RequestMapping("/testParam")
public String testParam(String username, String password){
    System.out.println("username:"+username+",password:"+password);
    return "success";
}
```

> 注：
>
> 若请求所传输的请求参数中有多个同名的请求参数，此时可以在控制器方法的形参中设置字符串数组或者字符串类型的形参接收此请求参数
>
> 若使用字符串数组类型的形参，此参数的数组中包含了每一个数据
>
> 若使用字符串类型的形参，此参数的值为每个数据中间使用逗号拼接的结果
>

#### 3、@RequestParam
@RequestParam是将请求参数和控制器方法的形参创建映射关系

@RequestParam注解一共有三个属性：

value：指定为形参赋值的请求参数的参数名

required：设置是否必须传输此请求参数，默认值为true

若设置为true时，则当前请求必须传输value所指定的请求参数，若没有传输该请求参数，且没有设置defaultValue属性，则页面报错400：Required String parameter ‘xxx’ is not present；若设置为false，则当前请求不是必须传输value所指定的请求参数，若没有传输，则注解所标识的形参的值为null

defaultValue：不管required属性值为true或false，当value所指定的请求参数没有传输或传输的值为""时，则使用默认值为形参赋值

#### 4、@RequestHeader
@RequestHeader是将请求头信息和控制器方法的形参创建映射关系

@RequestHeader注解一共有三个属性：value、required、defaultValue，用法同@RequestParam

#### 5、@CookieValue
@CookieValue是将cookie数据和控制器方法的形参创建映射关系

@CookieValue注解一共有三个属性：value、required、defaultValue，用法同@RequestParam

#### 6、通过POJO获取请求参数
可以在控制器方法的形参位置设置一个实体类类型的形参，此时若浏览器传输的请求参数的参数名和实体类中的属性名一致，那么请求参数就会为此属性赋值

``` html
<form th:action="@{/testpojo}" method="post">
  用户名：<input type="text" name="username"><br>
  密码：<input type="password" name="password"><br>
  性别：<input type="radio" name="sex" value="男">男<input type="radio" name="sex" value="女">女<br>
  年龄：<input type="text" name="age"><br>
  邮箱：<input type="text" name="email"><br>
  <input type="submit">
</form>
```

``` java
@RequestMapping("/testpojo")
public String testPOJO(User user){
    System.out.println(user);
    return "success";
}
//最终结果-->
//User{id=null, username='张三', password='123', age=23, sex='男', email='123@qq.com'}
```

#### 7、解决获取请求参数的乱码问题
解决获取请求参数的乱码问题，可以使用SpringMVC提供的编码过滤器CharacterEncodingFilter，但是必须在web.xml中进行注册

```xml
<!--配置springMVC的编码过滤器-->
<filter>
  <filter-name>CharacterEncodingFilter</filter-name>
  <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
  <init-param>
    <param-name>encoding</param-name>
    <param-value>UTF-8</param-value>
  </init-param>
  <init-param>
    <param-name>forceResponseEncoding</param-name>
    <param-value>true</param-value>
  </init-param>
</filter>
<filter-mapping>
  <filter-name>CharacterEncodingFilter</filter-name>
  <url-pattern>/*</url-pattern>
</filter-mapping>
```

> 注：
>
> SpringMVC中处理编码的过滤器一定要配置到其他过滤器之前，否则无效
>

## 五、域对象共享数据
#### 1、使用ServletAPI向request域对象共享数据
``` java
@RequestMapping("/testServletAPI")
public String testServletAPI(HttpServletRequest request){
request.setAttribute("testScope", "hello,servletAPI");
return "success";
}
```

#### 2、使用ModelAndView向request域对象共享数据
```java
@RequestMapping("/testModelAndView")
public ModelAndView testModelAndView(){
    /**
     * ModelAndView有Model和View的功能
     * Model主要用于向请求域共享数据
     * View主要用于设置视图，实现页面跳转
     */
    ModelAndView mav = new ModelAndView();
    //向请求域共享数据
    mav.addObject("testScope", "hello,ModelAndView");
    //设置视图，实现页面跳转
    mav.setViewName("success");
    return mav;
}
```

#### 3、使用Model向request域对象共享数据
```java
@RequestMapping("/testModel")
public String testModel(Model model){
    model.addAttribute("testScope", "hello,Model");
    return "success";
}
```

#### 4、使用map向request域对象共享数据
```java
@RequestMapping("/testMap")
public String testMap(Map<String, Object> map){
    map.put("testScope", "hello,Map");
    return "success";
}
```

#### 5、使用ModelMap向request域对象共享数据
```java
@RequestMapping("/testModelMap")
public String testModelMap(ModelMap modelMap){
    modelMap.addAttribute("testScope", "hello,ModelMap");
    return "success";
}
```

#### 6、Model、ModelMap、Map的关系
Model、ModelMap、Map类型的参数其实本质上都是 BindingAwareModelMap 类型的

```java
public interface Model{}
public class ModelMap extends LinkedHashMap<String, Object> {}
public class ExtendedModelMap extends ModelMap implements Model {}
public class BindingAwareModelMap extends ExtendedModelMap {}
```

#### 7、向session域共享数据
```java
@RequestMapping("/testSession")
public String testSession(HttpSession session){
    session.setAttribute("testSessionScope", "hello,session");
    return "success";
}
```

#### 8、向application域共享数据
```java
@RequestMapping("/testApplication")
public String testApplication(HttpSession session){
	ServletContext application = session.getServletContext();
    application.setAttribute("testApplicationScope", "hello,application");
    return "success";
}
```

## 六、SpringMVC的视图

SpringMVC中的视图是View接口，视图的作用渲染数据，将模型Model中的数据展示给用户

SpringMVC视图的种类很多，默认有转发视图和重定向视图

当工程引入jstl的依赖，转发视图会自动转换为JstlView

若使用的视图技术为Thymeleaf，在SpringMVC的配置文件中配置了Thymeleaf的视图解析器，由此视图解析器解析之后所得到的是ThymeleafView

#### 1、ThymeleafView
当控制器方法中所设置的视图名称没有任何前缀时，此时的视图名称会被SpringMVC配置文件中所配置的视图解析器解析，视图名称拼接视图前缀和视图后缀所得到的最终路径，会通过转发的方式实现跳转

```xml
@RequestMapping("/testHello")
public String testHello(){
    return "hello";
}
```

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1754593106486-c9f25d16-ba3a-43e5-8439-750b4846e599.png)

#### 2、转发视图
SpringMVC中默认的转发视图是InternalResourceView

SpringMVC中创建转发视图的情况：

当控制器方法中所设置的视图名称以"forward:"为前缀时，创建InternalResourceView视图，此时的视图名称不会被SpringMVC配置文件中所配置的视图解析器解析，而是会将前缀"forward:"去掉，剩余部分作为最终路径通过转发的方式实现跳转

例如"forward:/"，“forward:/employee”

```xml
@RequestMapping("/testForward")
public String testForward(){
    return "forward:/testHello";
}
```

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1754593106547-616c8490-ab55-4e50-852f-c0479b3f0af2.png)

#### 3、重定向视图
SpringMVC中默认的重定向视图是RedirectView

当控制器方法中所设置的视图名称以"redirect:"为前缀时，创建RedirectView视图，此时的视图名称不会被SpringMVC配置文件中所配置的视图解析器解析，而是会将前缀"redirect:"去掉，剩余部分作为最终路径通过重定向的方式实现跳转

例如"redirect:/"，“redirect:/employee”

```xml
@RequestMapping("/testRedirect")
public String testRedirect(){
    return "redirect:/testHello";
}
```

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1754593106564-4f8b547f-61fc-4e39-bb42-dddb62a5e4fa.png)

> 注：
>
> 重定向视图在解析时，会先将redirect:前缀去掉，然后会判断剩余部分是否以/开头，若是则会自动拼接上下文路径
>

#### 4、 转发和重定向的区别
<font style="color:rgba(0, 0, 0, 0.85) !important;">在 Web 开发中，**<font style="color:rgb(0, 0, 0) !important;">转发（Forward）**<font style="color:rgba(0, 0, 0, 0.85) !important;"> 和**<font style="color:rgb(0, 0, 0) !important;">重定向（Redirect）**<font style="color:rgba(0, 0, 0, 0.85) !important;"> 是两种页面跳转方式，核心区别在于跳转发生的 “位置” 和对客户端的影响不同，具体区别如下：

##### <font style="color:rgb(0, 0, 0);">1. 跳转发生的位置
+ **<font style="color:rgb(0, 0, 0) !important;">转发（服务器端跳转）**<font style="color:rgba(0, 0, 0, 0.85) !important;">  
<font style="color:rgba(0, 0, 0, 0.85) !important;">跳转发生在**<font style="color:rgb(0, 0, 0) !important;">服务器内部**<font style="color:rgba(0, 0, 0, 0.85) !important;">：客户端发送请求到服务器后，服务器内部将请求 “转发” 给另一个资源（Servlet、JSP 等），处理完成后直接将结果返回给客户端。  
<font style="color:rgba(0, 0, 0, 0.85) !important;">整个过程中，客户端**<font style="color:rgb(0, 0, 0) !important;">只发送一次请求**<font style="color:rgba(0, 0, 0, 0.85) !important;">，且不知道服务器内部的跳转逻辑。
+ **<font style="color:rgb(0, 0, 0) !important;">重定向（客户端跳转）**<font style="color:rgba(0, 0, 0, 0.85) !important;">  
<font style="color:rgba(0, 0, 0, 0.85) !important;">跳转由**<font style="color:rgb(0, 0, 0) !important;">客户端完成**<font style="color:rgba(0, 0, 0, 0.85) !important;">：服务器收到请求后，会返回一个特殊的响应（状态码 302），告诉客户端 “需要重新向新地址发送请求”。客户端收到响应后，会自动向新地址发送第二次请求。  
<font style="color:rgba(0, 0, 0, 0.85) !important;">整个过程中，客户端**<font style="color:rgb(0, 0, 0) !important;">发送两次请求**<font style="color:rgba(0, 0, 0, 0.85) !important;">，且明确知道跳转到了新地址。

##### <font style="color:rgb(0, 0, 0);">2. 浏览器地址栏变化
+ **<font style="color:rgb(0, 0, 0) !important;">转发**<font style="color:rgba(0, 0, 0, 0.85) !important;">：地址栏显示**<font style="color:rgb(0, 0, 0) !important;">最初请求的 URL**<font style="color:rgba(0, 0, 0, 0.85) !important;">，不会变化（因为客户端不知道服务器内部跳转）。
+ **<font style="color:rgb(0, 0, 0) !important;">重定向**<font style="color:rgba(0, 0, 0, 0.85) !important;">：地址栏显示**<font style="color:rgb(0, 0, 0) !important;">新的 URL**<font style="color:rgba(0, 0, 0, 0.85) !important;">（因为客户端向新地址发送了第二次请求）。

##### <font style="color:rgb(0, 0, 0);">3. 数据传递方式
+ **<font style="color:rgb(0, 0, 0) !important;">转发**<font style="color:rgba(0, 0, 0, 0.85) !important;">：可以通过`<font style="color:rgb(0, 0, 0);">request`<font style="color:rgba(0, 0, 0, 0.85) !important;">对象传递数据（如`<font style="color:rgb(0, 0, 0);">request.setAttribute()`<font style="color:rgba(0, 0, 0, 0.85) !important;">），因为整个过程共享同一个请求对象。
+ **<font style="color:rgb(0, 0, 0) !important;">重定向**<font style="color:rgba(0, 0, 0, 0.85) !important;">：**<font style="color:rgb(0, 0, 0) !important;">无法通过**`**<font style="color:rgb(0, 0, 0);">request**`**<font style="color:rgb(0, 0, 0) !important;">传递数据**<font style="color:rgba(0, 0, 0, 0.85) !important;">（两次请求是独立的，`<font style="color:rgb(0, 0, 0);">request`<font style="color:rgba(0, 0, 0, 0.85) !important;">对象会被销毁），需通过 URL 参数、Session 或数据库等方式传递数据。

##### <font style="color:rgb(0, 0, 0);">4. 适用场景
+ **<font style="color:rgb(0, 0, 0) !important;">转发**<font style="color:rgba(0, 0, 0, 0.85) !important;">：
    - <font style="color:rgba(0, 0, 0, 0.85) !important;">适合服务器内部资源跳转（如 Servlet 处理完数据后跳转到 JSP 展示结果）。
    - <font style="color:rgba(0, 0, 0, 0.85) !important;">需传递大量数据给目标页面时（通过`<font style="color:rgb(0, 0, 0);">request`<font style="color:rgba(0, 0, 0, 0.85) !important;">）。
    - <font style="color:rgba(0, 0, 0, 0.85) !important;">不希望用户看到跳转后的真实 URL 时。
+ **<font style="color:rgb(0, 0, 0) !important;">重定向**<font style="color:rgba(0, 0, 0, 0.85) !important;">：
    - <font style="color:rgba(0, 0, 0, 0.85) !important;">适合跳转到外部网站（如从自己的网站跳转到第三方登录页面）。
    - <font style="color:rgba(0, 0, 0, 0.85) !important;">防止表单重复提交（如用户提交表单后，重定向到结果页，避免刷新页面时重复提交）。
    - <font style="color:rgba(0, 0, 0, 0.85) !important;">希望用户明确知道地址变化时（如登录后跳转到首页）。

##### <font style="color:rgb(0, 0, 0);">5. 代码示例（Spring MVC）
+ **<font style="color:rgb(0, 0, 0) !important;">转发**<font style="color:rgba(0, 0, 0, 0.85) !important;">（服务器内部跳转）：

```xml
@GetMapping("/forward")
public String forwardDemo(HttpServletRequest request) {
    request.setAttribute("msg", "通过转发传递的数据");
    // 返回值以"forward:"开头，指定目标路径
    return "forward:/target"; 
}
```

+ **<font style="color:rgb(0, 0, 0) !important;">重定向**<font style="color:rgba(0, 0, 0, 0.85) !important;">（客户端跳转）：

```xml
@GetMapping("/redirect")
public String redirectDemo() {
    // 返回值以"redirect:"开头，指定目标路径
    return "redirect:/target"; 
}
```

##### <font style="color:rgb(0, 0, 0);">核心对比表

| **<font style="color:rgb(0, 0, 0) !important;">特性** | **<font style="color:rgb(0, 0, 0) !important;">转发（Forward）** | **<font style="color:rgb(0, 0, 0) !important;">重定向（Redirect）** |
| :--- | :--- | :--- |
| <font style="color:rgba(0, 0, 0, 0.85) !important;">跳转位置 | <font style="color:rgba(0, 0, 0, 0.85) !important;">服务器内部 | <font style="color:rgba(0, 0, 0, 0.85) !important;">客户端（二次请求） |
| <font style="color:rgba(0, 0, 0, 0.85) !important;">地址栏 | <font style="color:rgba(0, 0, 0, 0.85) !important;">不变 | <font style="color:rgba(0, 0, 0, 0.85) !important;">显示新地址 |
| <font style="color:rgba(0, 0, 0, 0.85) !important;">请求次数 | <font style="color:rgba(0, 0, 0, 0.85) !important;">1 次 | <font style="color:rgba(0, 0, 0, 0.85) !important;">2 次 |
| <font style="color:rgba(0, 0, 0, 0.85) !important;">数据传递 | <font style="color:rgba(0, 0, 0, 0.85) !important;">可通过`<font style="color:rgb(0, 0, 0);">request`<br/><font style="color:rgba(0, 0, 0, 0.85) !important;">传递 | <font style="color:rgba(0, 0, 0, 0.85) !important;">不可通过`<font style="color:rgb(0, 0, 0);">request`<br/><font style="color:rgba(0, 0, 0, 0.85) !important;">传递 |
| <font style="color:rgba(0, 0, 0, 0.85) !important;">适用场景 | <font style="color:rgba(0, 0, 0, 0.85) !important;">服务器内部资源跳转 | <font style="color:rgba(0, 0, 0, 0.85) !important;">外部链接、防重复提交等 |


  


<font style="color:rgba(0, 0, 0, 0.85) !important;">总结：转发是 “服务器帮客户端跳转”，重定向是 “服务器告诉客户端自己跳转”，根据是否需要暴露新地址、是否需要传递数据等场景选择使用。

#### 5、视图控制器view-controller
当控制器方法中，仅仅用来实现页面跳转，即只需要设置视图名称时，可以将处理器方法使用view-controller标签进行表示

```xml
<mvc:view-controller path="/testView" view-name="success"></mvc:view-controller>
```

> 注：
>
> 当SpringMVC中设置任何一个view-controller时，其他控制器中的请求映射将全部失效，此时需要在SpringMVC的核心配置文件中设置开启mvc注解驱动的标签：
>
> <mvc:annotation-driven />
>

## 七、RESTful
#### 1、RESTful简介
REST：**Re**presentational **S**tate **T**ransfer，表现层资源状态转移。

###### a>资源
资源是一种看待服务器的方式，即，将服务器看作是由很多离散的资源组成。每个资源是服务器上一个可命名的抽象概念。因为资源是一个抽象的概念，所以它不仅仅能代表服务器文件系统中的一个文件、数据库中的一张表等等具体的东西，可以将资源设计的要多抽象有多抽象，只要想象力允许而且客户端应用开发者能够理解。与面向对象设计类似，资源是以名词为核心来组织的，首先关注的是名词。一个资源可以由一个或多个URI来标识。URI既是资源的名称，也是资源在Web上的地址。对某个资源感兴趣的客户端应用，可以通过资源的URI与其进行交互。

###### b>资源的表述
资源的表述是一段对于资源在某个特定时刻的状态的描述。可以在客户端-服务器端之间转移（交换）。资源的表述可以有多种格式，例如HTML/XML/JSON/纯文本/图片/视频/音频等等。资源的表述格式可以通过协商机制来确定。请求-响应方向的表述通常使用不同的格式。

###### c>状态转移
状态转移说的是：在客户端和服务器端之间转移（transfer）代表资源状态的表述。通过转移和操作资源的表述，来间接实现操作资源的目的。

#### 2、RESTful的实现
具体说，就是 HTTP 协议里面，四个表示操作方式的动词：GET、POST、PUT、DELETE。

它们分别对应四种基本操作：GET 用来获取资源，POST 用来新建资源，PUT 用来更新资源，DELETE 用来删除资源。

REST 风格提倡 URL 地址使用统一的风格设计，从前到后各个单词使用斜杠分开，不使用问号键值对方式携带请求参数，而是将要发送给服务器的数据作为 URL 地址的一部分，以保证整体风格的一致性。

| 操作 | 传统方式 | REST风格 |
| --- | --- | --- |
| 查询操作 | getUserById?id=1 | user/1–>get请求方式 |
| 保存操作 | saveUser | user–>post请求方式 |
| 删除操作 | deleteUser?id=1 | user/1–>delete请求方式 |
| 更新操作 | updateUser | user–>put请求方式 |

#### 3、HiddenHttpMethodFilter
由于浏览器只支持发送get和post方式的请求，那么该如何发送put和delete请求呢？

SpringMVC 提供了 **HiddenHttpMethodFilter** 帮助我们**将 POST 请求转换为 DELETE 或 PUT 请求**

**HiddenHttpMethodFilter** 处理put和delete请求的条件：

a>当前请求的请求方式必须为post

b>当前请求必须传输请求参数_method

满足以上条件，**HiddenHttpMethodFilter** 过滤器就会将当前请求的请求方式转换为请求参数_method的值，因此请求参数_method的值才是最终的请求方式

在web.xml中注册**HiddenHttpMethodFilter**

```xml
<filter>
  <filter-name>HiddenHttpMethodFilter</filter-name>
  <filter-class>org.springframework.web.filter.HiddenHttpMethodFilter</filter-class>
</filter>
<filter-mapping>
  <filter-name>HiddenHttpMethodFilter</filter-name>
  <url-pattern>/*</url-pattern>
</filter-mapping>
```

> 注：
>
> 目前为止，SpringMVC中提供了两个过滤器：CharacterEncodingFilter和HiddenHttpMethodFilter
>
> 在web.xml中注册时，必须先注册CharacterEncodingFilter，再注册HiddenHttpMethodFilter
>
> 原因：
>
> + 在 CharacterEncodingFilter 中通过 request.setCharacterEncoding(encoding) 方法设置字符集的
> + request.setCharacterEncoding(encoding) 方法要求前面不能有任何获取请求参数的操作
> + 而 HiddenHttpMethodFilter 恰恰有一个获取请求方式的操作：
>

## 八、RESTful案例
#### 1、准备工作
和传统 CRUD 一样，实现对员工信息的增删改查。

+ 搭建环境
+ 准备实体类

```java
package com.atguigu.mvc.bean;

public class Employee {

    private Integer id;
    private String lastName;

    private String email;

    private Integer gender;

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getLastName() {
        return lastName;
    }

    public void setLastName(String lastName) {
        this.lastName = lastName;
    }

    public String getEmail() {
        return email;
    }

    public void setEmail(String email) {
        this.email = email;
    }

    public Integer getGender() {
        return gender;
    }

    public void setGender(Integer gender) {
        this.gender = gender;
    }

    public Employee(Integer id, String lastName, String email, Integer gender) {
        super();
        this.id = id;
        this.lastName = lastName;
        this.email = email;
        this.gender = gender;
    }

    public Employee() {
    }
}
```

+ 准备dao模拟数据

```xml
package com.atguigu.mvc.dao;

import java.util.Collection;
import java.util.HashMap;
import java.util.Map;

import com.atguigu.mvc.bean.Employee;
import org.springframework.stereotype.Repository;


@Repository
public class EmployeeDao {

    private static Map<Integer, Employee> employees = null;

    static{
        employees = new HashMap<Integer, Employee>();

        employees.put(1001, new Employee(1001, "E-AA", "aa@163.com", 1));
        employees.put(1002, new Employee(1002, "E-BB", "bb@163.com", 1));
        employees.put(1003, new Employee(1003, "E-CC", "cc@163.com", 0));
        employees.put(1004, new Employee(1004, "E-DD", "dd@163.com", 0));
        employees.put(1005, new Employee(1005, "E-EE", "ee@163.com", 1));
    }

    private static Integer initId = 1006;

    public void save(Employee employee){
        if(employee.getId() == null){
            employee.setId(initId++);
        }
        employees.put(employee.getId(), employee);
    }

    public Collection<Employee> getAll(){
        return employees.values();
    }

    public Employee get(Integer id){
        return employees.get(id);
    }

    public void delete(Integer id){
        employees.remove(id);
    }
}
```

#### 2、功能清单
| 功能 | URL 地址 | 请求方式 |
| --- | --- | --- |
| 访问首页√ | / | GET |
| 查询全部数据√ | /employee | GET |
| 删除√ | /employee/2 | DELETE |
| 跳转到添加数据页面√ | /toAdd | GET |
| 执行保存√ | /employee | POST |
| 跳转到更新数据页面√ | /employee/2 | GET |
| 执行更新√ | /employee | PUT |


#### 3、具体功能：访问首页
###### a>配置view-controller
```xml
<mvc:view-controller path="/" view-name="index"/>
```

###### b>创建页面
```xml
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
  <head>
    <meta charset="UTF-8" >
    <title>Title</title>
  </head>
  <body>
    <h1>首页</h1>
    <a th:href="@{/employee}">访问员工信息</a>
  </body>
</html>
```

#### 4、具体功能：查询所有员工数据
###### a>控制器方法
```xml
/**
 * 处理获取员工列表的GET请求
 * @param model Spring MVC模型对象，用于向视图传递数据
 * @return 要渲染的视图名称
 */
@RequestMapping(value = "/employee", method = RequestMethod.GET)
public String getEmployeeList(Model model) {
    // 1. 从DAO层获取所有员工数据
    Collection<Employee> employeeList = employeeDao.getAll();
    
    // 2. 将员工列表添加到模型对象中，属性名为"employeeList"
    model.addAttribute("employeeList", employeeList);
    
    // 3. 返回视图名称"employee_list"
    //    视图解析器会根据配置找到对应的视图文件(如employee_list.jsp)
    return "employee_list";
}
```

1. `**@RequestMapping**`** 注解**：
    - `**value = "/employee"**`：映射的URL路径
    - `**method = RequestMethod.GET**`：只处理HTTP GET请求
2. `**Model**`** 参数**：
    - Spring MVC提供的模型对象
    - 用于控制器向视图传递数据
    - 添加的属性可以在视图中通过EL表达式访问（如`**${employeeList}**`）
3. **返回值**：
    - 返回字符串表示逻辑视图名称
    - 视图解析器会将其解析为实际视图（如JSP、Thymeleaf模板等）

###### b>创建employee_list.html
```xml
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
  <head>
    <meta charset="UTF-8">
    <title>Employee Info</title>
    <script type="text/javascript" th:src="@{/static/js/vue.js}"></script>
  </head>
  <body>

    <table border="1" cellpadding="0" cellspacing="0" style="text-align: center;" id="dataTable">
      <tr>
        <th colspan="5">Employee Info</th>
      </tr>
      <tr>
        <th>id</th>
        <th>lastName</th>
        <th>email</th>
        <th>gender</th>
        <th>options(<a th:href="@{/toAdd}">add</a>)</th>
      </tr>
      <tr th:each="employee : ${employeeList}">
        <td th:text="${employee.id}"></td>
        <td th:text="${employee.lastName}"></td>
        <td th:text="${employee.email}"></td>
        <td th:text="${employee.gender}"></td>
        <td>
          <a class="deleteA" @click="deleteEmployee" th:href="@{'/employee/'+${employee.id}}">delete</a>
          <a th:href="@{'/employee/'+${employee.id}}">update</a>
        </td>
      </tr>
    </table>
  </body>
</html>
```

#### 5、具体功能：删除
###### a>创建处理delete请求方式的表单
```xml
<form id="delete_form" method="post">
  <!--通过隐藏字段 _method 来"伪造"其他 HTTP 方法（如 DELETE、PUT）-->
  <input type="hidden" name="_method" value="delete"/>
</form>
```

###### b>删除超链接绑定点击事件
引入vue.js

```xml
<script type="text/javascript" th:src="@{/static/js/vue.js}"></script>
```

删除超链接

```xml
<a class="deleteA" @click="deleteEmployee" th:href="@{'/employee/'+${employee.id}}">delete</a>
```

通过vue处理点击事件

```xml
<script type="text/javascript">
  // 创建Vue实例，挂载到id为"dataTable"的DOM元素上
  var vue = new Vue({
    el: "#dataTable",
    methods: {
      // 定义删除员工的方法
      deleteEmployee: function(event) {
        // 1. 获取delete_form表单元素
        var delete_form = document.getElementById("delete_form");
        
        // 2. 将表单的action设置为触发事件的元素的href属性
        // (通常是<a>标签的链接地址)
        delete_form.action = event.target.href;
        
        // 3. 提交表单
        delete_form.submit();
        
        // 4. 阻止默认事件行为(防止链接直接跳转)
        event.preventDefault();
      }
    }
  });
</script>
```

###### c>控制器方法
```xml
@RequestMapping(value = "/employee/{id}", method = RequestMethod.DELETE)
public String deleteEmployee(@PathVariable("id") Integer id){
    employeeDao.delete(id);
    return "redirect:/employee";
}
```

#### 6、具体功能：跳转到添加数据页面
###### a>配置view-controller
```xml
<mvc:view-controller path="/toAdd" view-name="employee_add"></mvc:view-controller>
```

###### b>创建employee_add.html
```xml
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
  <head>
    <meta charset="UTF-8">
    <title>Add Employee</title>
  </head>
  <body>

    <form th:action="@{/employee}" method="post">
      lastName:<input type="text" name="lastName"><br>
      email:<input type="text" name="email"><br>
      gender:<input type="radio" name="gender" value="1">male
      <input type="radio" name="gender" value="0">female<br>
      <input type="submit" value="add"><br>
    </form>

  </body>
</html>
```

#### 7、具体功能：执行保存
###### a>控制器方法
```xml
@RequestMapping(value = "/employee", method = RequestMethod.POST)
public String addEmployee(Employee employee){
employeeDao.save(employee);
return "redirect:/employee";
}
```

#### 8、具体功能：跳转到更新数据页面
###### a>修改超链接
```xml
<a th:href="@{'/employee/'+${employee.id}}">update</a>
```

###### b>控制器方法
```xml
@RequestMapping(value = "/employee/{id}", method = RequestMethod.GET)
public String getEmployeeById(@PathVariable("id") Integer id, Model model){
    Employee employee = employeeDao.get(id);
    model.addAttribute("employee", employee);
    return "employee_update";
}
```

###### c>创建employee_update.html
```xml
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
  <head>
    <meta charset="UTF-8">
    <title>Update Employee</title>
  </head>
  <body>

    <form th:action="@{/employee}" method="post">
      <input type="hidden" name="_method" value="put">
      <input type="hidden" name="id" th:value="${employee.id}">
      lastName:<input type="text" name="lastName" th:value="${employee.lastName}"><br>
      email:<input type="text" name="email" th:value="${employee.email}"><br>

      gender:<input type="radio" name="gender" value="1" th:field="${employee.gender}">male
      <input type="radio" name="gender" value="0" th:field="${employee.gender}">female<br>
      <input type="submit" value="update"><br>
    </form>

  </body>
</html>
```

#### 9、具体功能：执行更新
###### a>控制器方法
```xml
/**
 * 处理更新员工信息的PUT请求
 * @param employee 从请求参数自动绑定的Employee对象
 * @return 重定向到员工列表页面
 */
@RequestMapping(value = "/employee", method = RequestMethod.PUT)
public String updateEmployee(Employee employee) {
    // 1. 调用DAO层保存/更新员工信息
    employeeDao.save(employee);
    
    // 2. 重定向到员工列表页面
    //    "redirect:"前缀会告诉Spring MVC进行重定向而非视图解析
    return "redirect:/employee";
}
```

## 八、HttpMessageConverter
HttpMessageConverter，报文信息转换器，将请求报文转换为Java对象，或将Java对象转换为响应报文

HttpMessageConverter提供了两个注解和两个类型：@RequestBody，@ResponseBody，RequestEntity，

ResponseEntity

#### 1、@RequestBody
@RequestBody可以获取请求体，需要在控制器方法设置一个形参，使用@RequestBody进行标识，当前请求的请求体就会为当前注解所标识的形参赋值

```xml
<form th:action="@{/testRequestBody}" method="post">
  用户名：<input type="text" name="username"><br>
  密码：<input type="password" name="password"><br>
  <input type="submit">
</form>
```

```xml
@RequestMapping("/testRequestBody")
public String testRequestBody(@RequestBody String requestBody){
    System.out.println("requestBody:"+requestBody);
    return "success";
}
```

输出结果：

requestBody:username=admin&password=123456

#### 2、RequestEntity
RequestEntity封装请求报文的一种类型，需要在控制器方法的形参中设置该类型的形参，当前请求的请求报文就会赋值给该形参，可以通过getHeaders()获取请求头信息，通过getBody()获取请求体信息

```xml
@RequestMapping("/testRequestEntity")
public String testRequestEntity(RequestEntity<String> requestEntity){
    System.out.println("requestHeader:"+requestEntity.getHeaders());
    System.out.println("requestBody:"+requestEntity.getBody());
    return "success";
}
```

输出结果：  
requestHeader:[host:“localhost:8080”, connection:“keep-alive”, content-length:“27”, cache-control:“max-age=0”, sec-ch-ua:“” Not A;Brand";v=“99”, “Chromium”;v=“90”, “Google Chrome”;v=“90"”, sec-ch-ua-mobile:“?0”, upgrade-insecure-requests:“1”, origin:“http://localhost:8080”, user-agent:“Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/90.0.4430.93 Safari/537.36”]  
requestBody:username=admin&password=123

#### 3、@ResponseBody
@ResponseBody用于标识一个控制器方法，可以将该方法的返回值直接作为响应报文的响应体响应到浏览器

```xml
@RequestMapping("/testResponseBody")
@ResponseBody
public String testResponseBody(){
    return "success";
}
```

这段代码的作用：

1. 当访问 `**/testResponseBody**` 路径时
2. 方法返回字符串 `**"success"**`
3. `**@ResponseBody**` 注解确保这个字符串直接作为响应体返回
4. 浏览器将收到纯文本内容 `**"success"**`，而不是跳转到一个名为 "success" 的视图

#### 4、SpringMVC处理json
@ResponseBody处理json的步骤：

a>导入jackson的依赖

```xml
<dependency>
  <groupId>com.fasterxml.jackson.core</groupId>
  <artifactId>jackson-databind</artifactId>
  <version>2.12.1</version>
</dependency>
```

b>在SpringMVC的核心配置文件中开启mvc的注解驱动，此时在HandlerAdaptor中会自动装配一个消息转换器：MappingJackson2HttpMessageConverter，可以将响应到浏览器的Java对象转换为Json格式的字符串

```xml
<mvc:annotation-driven />
```

c>在处理器方法上使用@ResponseBody注解进行标识

d>将Java对象直接作为控制器方法的返回值返回，就会自动转换为Json格式的字符串

```xml
@RequestMapping("/testResponseUser")
@ResponseBody
public User testResponseUser(){
    return new User(1001,"admin","123456",23,"男");
}
```

浏览器的页面中展示的结果：

{“id”:1001,“username”:“admin”,“password”:“123456”,“age”:23,“sex”:“男”}

#### 5、SpringMVC处理ajax
a>请求超链接：

```xml
<div id="app">
	<a th:href="@{/testAjax}" @click="testAjax">testAjax</a><br>
</div>
```

b>通过vue和axios处理点击事件：

```xml
<script type="text/javascript" th:src="@{/static/js/vue.js}"></script>
<script type="text/javascript" th:src="@{/static/js/axios.min.js}"></script>
<script type="text/javascript">
    var vue = new Vue({
        el:"#app",
        methods:{
            testAjax:function (event) {
                axios({
                    method:"post",
                    url:event.target.href,
                    params:{
                        username:"admin",
                        password:"123456"
                    }
                }).then(function (response) {
                    alert(response.data);
                });
                event.preventDefault();
            }
        }
    });
</script>
```

c>控制器方法：

```xml
@RequestMapping("/testAjax")
@ResponseBody
public String testAjax(String username, String password){
    System.out.println("username:"+username+",password:"+password);
    return "hello,ajax";
}
```

#### 6、@RestController注解
@RestController注解是springMVC提供的一个复合注解，标识在控制器的类上，就相当于为类添加了@Controller注解，并且为其中的每个方法添加了@ResponseBody注解

#### 7、ResponseEntity
ResponseEntity用于控制器方法的返回值类型，该控制器方法的返回值就是响应到浏览器的响应报文

## 九、文件上传和下载
#### 1、文件下载
使用ResponseEntity实现下载文件的功能

```xml
/**
 * 处理文件下载请求
 * @param session HttpSession对象，用于获取ServletContext
 * @return 包含文件数据的ResponseEntity
 * @throws IOException 如果文件读取失败
 */
@RequestMapping("/testDown")
public ResponseEntity<byte[]> testResponseEntity(HttpSession session) throws IOException {
    // 1. 获取ServletContext对象（用于访问web应用资源）
    ServletContext servletContext = session.getServletContext();
    
    // 2. 获取服务器上文件的真实路径
    // "/static/img/1.jpg"是web应用中的相对路径
    String realPath = servletContext.getRealPath("/static/img/1.jpg");
    
    // 3. 创建文件输入流
    InputStream is = new FileInputStream(realPath);
    
    // 4. 创建与文件大小相同的字节数组
    byte[] bytes = new byte[is.available()];
    
    // 5. 将文件内容读取到字节数组中
    is.read(bytes);
    
    // 6. 设置响应头信息
    MultiValueMap<String, String> headers = new HttpHeaders();
    // Content-Disposition头告诉浏览器以附件形式处理响应
    // "attachment"表示下载，"filename=1.jpg"指定下载文件名
    headers.add("Content-Disposition", "attachment;filename=1.jpg");
    
    // 7. 设置HTTP状态码（200表示成功）
    HttpStatus statusCode = HttpStatus.OK;
    
    // 8. 创建ResponseEntity对象
    // 包含：文件数据字节数组、响应头、状态码
    ResponseEntity<byte[]> responseEntity = new ResponseEntity<>(bytes, headers, statusCode);
    
    // 9. 关闭输入流
    is.close();
    
    // 10. 返回响应实体
    return responseEntity;
}
```

#### 2、文件上传
文件上传要求form表单的请求方式必须为post，并且添加属性enctype=“multipart/form-data”

SpringMVC中将上传的文件封装到MultipartFile对象中，通过此对象可以获取文件相关信息

上传步骤：

a>在SpringMVC的配置文件中添加配置(Spring6)：

```xml
<bean id="multipartResolver" class="org.springframework.web.multipart.support.StandardServletMultipartResolver"/>
```

a>添加依赖：

```xml
<dependency>
  <groupId>commons-fileupload</groupId>
  <artifactId>commons-fileupload</artifactId>
  <version>1.3.1</version>
</dependency>
```

b>在SpringMVC的配置文件中添加配置：

```xml
<bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver"></bean>
```

c>控制器方法：

```xml
/**
 * 处理文件上传请求
 * @param photo 上传的文件，会自动绑定到MultipartFile对象
 * @param session HttpSession对象，用于获取ServletContext
 * @return 上传成功后的跳转页面
 * @throws IOException 如果文件操作失败
 */
@RequestMapping("/testUp")
public String testUp(@RequestParam("photo") MultipartFile photo, HttpSession session) throws IOException {
    
    // 1. 获取原始文件名
    String fileName = photo.getOriginalFilename();
    
    // 2. 处理文件名：生成随机文件名并保留扩展名
    // 获取文件扩展名（如.jpg）
    String hzName = fileName.substring(fileName.lastIndexOf("."));
    // 使用UUID生成随机文件名，避免文件名冲突
    fileName = UUID.randomUUID().toString() + hzName;
    
    // 3. 获取服务器上存储文件的目录路径
    ServletContext servletContext = session.getServletContext();
    // "photo"是web应用下的相对目录
    String photoPath = servletContext.getRealPath("photo");
    File file = new File(photoPath);
    
    // 4. 如果目录不存在，则创建
    if(!file.exists()){
        file.mkdir(); // 创建单级目录
    }
    
    // 5. 拼接最终文件存储路径
    String finalPath = photoPath + File.separator + fileName;
    
    // 6. 将上传的文件转存到服务器指定位置
    photo.transferTo(new File(finalPath));
    
    // 7. 返回成功页面
    return "success";
}
```

## 十、拦截器
#### 1、拦截器的配置

SpringMVC中的拦截器用于拦截控制器方法的执行

SpringMVC中的拦截器需要实现HandlerInterceptor

SpringMVC的拦截器必须在SpringMVC的配置文件中进行配置：

```xml
<bean class="com.atguigu.interceptor.FirstInterceptor"></bean>
<ref bean="firstInterceptor"></ref>

<mvc:interceptor>
    <!-- 指定拦截的路径 -->
    <mvc:mapping path="/**"/>
    
    <!-- 指定排除的路径 -->
    <mvc:exclude-mapping path="/testRequestEntity"/>
    
    <!-- 引用拦截器Bean -->
    <ref bean="firstInterceptor"/>
</mvc:interceptor>
```

#### 2、拦截器的三个抽象方法
SpringMVC中的拦截器有三个抽象方法：

+ **preHandle**：控制器方法执行之前执行preHandle()，其boolean类型的返回值表示是否拦截或放行，返回true为放行，即调用控制器方法；返回false表示拦截，即不调用控制器方法
+ **postHandle**：控制器方法执行之后执行postHandle()
+ **afterComplation**：处理完视图和模型数据，渲染视图完毕之后执行afterComplation()

#### 3、多个拦截器的执行顺序
**a>若每个拦截器的preHandle()都返回true**

+ 此时多个拦截器的执行顺序和拦截器在SpringMVC的配置文件的配置顺序有关：preHandle()会按照配置的顺序执行，而postHandle()和afterComplation()会按照配置的反序执行

**b>若某个拦截器的preHandle()返回了false**

+ preHandle()返回false和它之前的拦截器的preHandle()都会执行，postHandle()都不执行，返回false的拦截器之前的拦截器的afterComplation()会执行

## 十一、异常处理器
#### 1、基于配置的异常处理
SpringMVC提供了一个处理控制器方法执行过程中所出现的异常的接口：HandlerExceptionResolver

HandlerExceptionResolver接口的实现类有：DefaultHandlerExceptionResolver和SimpleMappingExceptionResolver

SpringMVC提供了自定义的异常处理器SimpleMappingExceptionResolver，使用方式：

```xml
<bean class="org.springframework.web.servlet.handler.SimpleMappingExceptionResolver">
  <!-- 
    SimpleMappingExceptionResolver 是Spring MVC提供的异常解析器，
    用于将应用程序抛出的异常映射到特定的错误视图
  -->
  
  <property name="exceptionMappings">
    <props>
      <!-- 
        exceptionMappings 属性定义异常类到视图名的映射关系
        使用<props>标签配置，其中每个<prop>表示一个映射
        key: 完全限定的异常类名
        value: 对应的视图名称(不需要文件扩展名)
      -->
      <prop key="java.lang.ArithmeticException">error</prop>
      <!-- 
        示例: 当发生ArithmeticException(算术异常，如除以零)时，
        将会转发到名为"error"的视图
        可以添加多个<prop>元素来映射不同的异常
      -->
    </props>
  </property>

  <property name="order" value="0"/>
  <!-- 值越小优先级越高 -->
  <property name="exceptionAttribute" value="ex"></property>
  <!-- 
    exceptionAttribute 属性指定将异常对象存入请求属性时使用的属性名
    默认值是"exception"，这里设置为"ex"
    这样在错误视图中可以通过${ex}访问异常对象
  -->
</bean>
```

#### 2、基于注解的异常处理
```java
@ControllerAdvice
public class ExceptionController {
    /**
     * 处理 ArithmeticException 异常
     * @param ex 捕获到的异常对象
     * @param model Spring MVC 模型对象，用于向视图传递数据
     * @return 要渲染的视图名称
     */
    @ExceptionHandler(ArithmeticException.class)
    public String handleArithmeticException(Exception ex, Model model) {
        // 将异常对象添加到模型，属性名为"ex"
        model.addAttribute("ex", ex);

        // 返回错误视图名称（对应error.jsp或error.html等）
        return "error";
        
    }
}
```

## 十二、注解配置SpringMVC

使用配置类和注解代替web.xml和SpringMVC配置文件的功能

#### 1、创建初始化类，代替web.xml
在Servlet3.0环境中，容器会在类路径中查找实现javax.servlet.ServletContainerInitializer接口的类，如果找到的话就用它来配置Servlet容器。  
Spring提供了这个接口的实现，名为SpringServletContainerInitializer，这个类反过来又会查找实现WebApplicationInitializer的类并将配置的任务交给它们来完成。Spring3.2引入了一个便利的WebApplicationInitializer基础实现，名为AbstractAnnotationConfigDispatcherServletInitializer，当我们的类扩展了AbstractAnnotationConfigDispatcherServletInitializer并将其部署到Servlet3.0容器的时候，容器会自动发现它，并用它来配置Servlet上下文。

```java
/**
 * Web应用程序初始化类，继承自AbstractAnnotationConfigDispatcherServletInitializer。
 * 这是Spring MVC基于Java配置的Web应用初始化的标准方式，替代了传统的web.xml配置。
 */
public class WebInit extends AbstractAnnotationConfigDispatcherServletInitializer {

    /**
     * 获取根应用上下文的配置类。
     * 这些配置类会创建根Spring应用上下文，通常包含服务层、数据层等非Web组件的配置。
     * 
     * @return 根配置类的数组，这里返回SpringConfig类
     */
    @Override
    protected Class<?>[] getRootConfigClasses() {
        return new Class[]{SpringConfig.class};
    }

    /**
     * 获取Servlet应用上下文的配置类。
     * 这些配置类会创建DispatcherServlet的应用上下文，通常包含控制器、视图解析器等Web相关组件的配置。
     * 
     * @return Servlet配置类的数组，这里返回WebConfig类
     */
    @Override
    protected Class<?>[] getServletConfigClasses() {
        return new Class[]{WebConfig.class};
    }

    /**
     * 指定DispatcherServlet的映射路径。
     * 这里配置为"/"，表示DispatcherServlet将处理所有的请求（除了静态资源，这通常在WebConfig中配置）。
     * 
     * @return Servlet映射路径的数组
     */
    @Override
    protected String[] getServletMappings() {
        return new String[]{"/"};
    }

    /**
     * 注册Servlet过滤器。
     * 这些过滤器会被自动添加到Servlet上下文中，并按返回数组的顺序执行。
     * 
     * @return 过滤器数组，这里配置了两个常用过滤器：
     *         1. CharacterEncodingFilter - 设置请求和响应的字符编码为UTF-8
     *         2. HiddenHttpMethodFilter - 支持通过隐藏字段(_method)实现RESTful风格的HTTP方法(PUT,DELETE等)
     */
    @Override
    protected Filter[] getServletFilters() {
        // 创建字符编码过滤器并配置UTF-8编码
        CharacterEncodingFilter encodingFilter = new CharacterEncodingFilter();
        encodingFilter.setEncoding("UTF-8"); // 设置编码为UTF-8
        encodingFilter.setForceRequestEncoding(true); // 强制请求使用指定编码

        // 创建HTTP方法过滤器，支持RESTful风格的PUT、DELETE等请求
        HiddenHttpMethodFilter hiddenHttpMethodFilter = new HiddenHttpMethodFilter();

        return new Filter[]{encodingFilter, hiddenHttpMethodFilter};
    }
}
```

###### 补充说明
1. **AbstractAnnotationConfigDispatcherServletInitializer** 是Spring提供的一个便捷基类，用于基于Java配置初始化Spring MVC应用。
2. **根上下文和Servlet上下文**：
    - 根上下文(root context)通常包含服务层、数据层等bean
    - Servlet上下文(servlet context)包含Web相关的bean(控制器、视图解析器等)
    - Servlet上下文可以访问根上下文中的bean，但反之不行
3. **过滤器说明**：
    - `**CharacterEncodingFilter**`确保请求和响应使用UTF-8编码，解决中文乱码问题
    - `**HiddenHttpMethodFilter**`允许HTML表单通过隐藏的_method参数发送PUT、DELETE等请求
4. 这种配置方式完全替代了传统的web.xml配置，是现代Spring应用推荐的方式。

#### 2、创建SpringConfig配置类，代替spring的配置文件
```java
@Configuration
public class SpringConfig {

}
```

#### 3、创建WebConfig配置类，代替SpringMVC的配置文件
```java
/**
 * Spring MVC Web配置类，用于配置Web相关的组件。
 * 这个类会被DispatcherServlet加载，用于创建Web应用上下文。
 */
@Configuration  // 标识这是一个Spring配置类
@ComponentScan("com.atguigu.mvc.controller")  // 自动扫描指定包下的组件(如@Controller)
@EnableWebMvc  // 启用Spring MVC注解驱动，相当于XML配置中的<mvc:annotation-driven>
public class WebConfig implements WebMvcConfigurer {

    /**
     * 配置默认Servlet处理。
     * 启用后，DispatcherServlet会将无法处理的请求转发给默认Servlet(如Tomcat的DefaultServlet)，
     * 主要用于处理静态资源。
     *
     * @param configurer 默认Servlet处理器配置器
     */
    @Override
    public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) {
        configurer.enable();  // 启用默认Servlet处理
    }

    /**
     * 配置多部分文件上传解析器。
     * 用于处理文件上传请求(enctype="multipart/form-data")。
     *
     * @return CommonsMultipartResolver实例
     */
    @Bean
    public CommonsMultipartResolver multipartResolver() {
        return new CommonsMultipartResolver();  // 创建并返回文件上传解析器
    }

    /**
     * 添加拦截器配置。
     * 可以注册一个或多个拦截器，并指定拦截路径。
     *
     * @param registry 拦截器注册表
     */
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        FirstInterceptor firstInterceptor = new FirstInterceptor();  // 创建拦截器实例
        registry.addInterceptor(firstInterceptor)  // 注册拦截器
        .addPathPatterns("/**");  // 指定拦截所有路径
    }

    /**
     * 配置Thymeleaf模板解析器。
     * 负责定位模板文件，设置模板前缀、后缀等属性。
     *
     * @return ITemplateResolver实例
     */
    @Bean
    public ITemplateResolver templateResolver() {
        // 获取当前Web应用上下文
        WebApplicationContext webApplicationContext = ContextLoader.getCurrentWebApplicationContext();

        // 创建Servlet上下文模板解析器
        ServletContextTemplateResolver templateResolver = new ServletContextTemplateResolver(
            webApplicationContext.getServletContext());
        templateResolver.setPrefix("/WEB-INF/templates/");  // 设置模板前缀路径
        templateResolver.setSuffix(".html");  // 设置模板后缀
        templateResolver.setCharacterEncoding("UTF-8");  // 设置字符编码
        templateResolver.setTemplateMode(TemplateMode.HTML);  // 设置模板模式为HTML
        return templateResolver;
    }

    /**
     * 配置Thymeleaf模板引擎。
     * 将模板解析器与模板引擎关联。
     *
     * @param templateResolver 模板解析器
     * @return SpringTemplateEngine实例
     */
    @Bean
    public SpringTemplateEngine templateEngine(ITemplateResolver templateResolver) {
        SpringTemplateEngine templateEngine = new SpringTemplateEngine();
        templateEngine.setTemplateResolver(templateResolver);  // 设置模板解析器
        return templateEngine;
    }

    /**
     * 配置视图解析器。
     * 将模板引擎与视图解析器关联，用于解析视图名称到实际视图。
     *
     * @param templateEngine 模板引擎
     * @return ViewResolver实例
     */
    @Bean
    public ViewResolver viewResolver(SpringTemplateEngine templateEngine) {
        ThymeleafViewResolver viewResolver = new ThymeleafViewResolver();
        viewResolver.setCharacterEncoding("UTF-8");  // 设置视图编码
        viewResolver.setTemplateEngine(templateEngine);  // 设置模板引擎
        return viewResolver;
    }
}
```

##### 补充说明
###### 核心注解：
    - `@Configuration`：标识这是一个Spring配置类
    - `**@ComponentScan**`：自动扫描指定包下的Spring组件（如`**@Controller**`）
    - `**@EnableWebMvc**`：启用Spring MVC的默认配置，包括：
        * 支持`**@Controller**`注解
        * 注册`**RequestMappingHandlerMapping**`和`**RequestMappingHandlerAdapter**`
        * 支持类型转换和验证
        * 消息转换器（如JSON转换）

###### WebMvcConfigurer接口：
    - 提供回调方法来自定义Spring MVC配置
    - 可以配置拦截器、视图解析器、消息转换器等

###### Thymeleaf配置：
    - 三件套配置：模板解析器 → 模板引擎 → 视图解析器
    - 支持HTML模板渲染，模板文件存放在`**/WEB-INF/templates/**`目录下

###### 文件上传：
    - `**CommonsMultipartResolver**`是基于Apache Commons FileUpload的文件上传处理器

###### 拦截器：
    - 可以添加多个拦截器，每个拦截器可以指定不同的拦截路径
    - 适用于权限检查、日志记录等横切关注点

###### 静态资源处理：
    - `**configureDefaultServletHandling**`启用后，静态资源请求会由Servlet容器默认Servlet处理
    - 另一种方式是使用`**addResourceHandlers**`方法来明确指定静态资源位置

#### 4、为什么不再需要 web.xml 和 SpringMVC.xml 配置
现代 Spring 应用（特别是 Spring Boot 和基于 Java 配置的 Spring MVC）已经不再需要传统的 `**web.xml**` 和 `**SpringMVC.xml**` 配置文件，原因如下：

##### 1. 替代 web.xml 的机制
###### 传统方式 (web.xml)
```xml
<!-- web.xml 示例 -->
<web-app>
  <servlet>
    <servlet-name>dispatcher</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <init-param>
      <param-name>contextConfigLocation</param-name>
      <param-value>/WEB-INF/springmvc.xml</param-value>
    </init-param>
    <load-on-startup>1</load-on-startup>
  </servlet>

  <servlet-mapping>
    <servlet-name>dispatcher</servlet-name>
    <url-pattern>/</url-pattern>
  </servlet-mapping>

  <filter>
    <filter-name>encodingFilter</filter-name>
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
    <init-param>
      <param-name>encoding</param-name>
      <param-value>UTF-8</param-value>
    </init-param>
  </filter>
</web-app>
```

###### 现代替代方案
你的 `**WebInit**` 类通过以下方式替代了 web.xml：

```xml
public class WebInit extends AbstractAnnotationConfigDispatcherServletInitializer {
    // 替代了 servlet 和 servlet-mapping 配置
    @Override protected String[] getServletMappings() { return new String[]{"/"}; }

    // 替代了 filter 配置
    @Override protected Filter[] getServletFilters() { return new Filter[]{...}; }

    // 替代了 contextConfigLocation 参数
    @Override protected Class<?>[] getRootConfigClasses() { return ...; }
    @Override protected Class<?>[] getServletConfigClasses() { return ...; }
}
```

**原理**：

+ Servlet 3.0+ 规范引入了 `**ServletContainerInitializer**` 接口
+ Spring 提供了 `**SpringServletContainerInitializer**` 实现
+ 你的 `**WebInit**` 会被自动检测并用于初始化 Web 应用

##### 2. 替代 SpringMVC.xml 的机制
###### 传统方式 (SpringMVC.xml)
```xml
<!-- SpringMVC.xml 示例 -->
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:mvc="http://www.springframework.org/schema/mvc"
  xmlns:context="http://www.springframework.org/schema/context">

  <mvc:annotation-driven/>
  <context:component-scan base-package="com.atguigu.mvc.controller"/>

  <mvc:default-servlet-handler/>

  <bean id="multipartResolver" class="...CommonsMultipartResolver"/>

  <!-- Thymeleaf 配置 -->
  <bean id="templateResolver" class="...ServletContextTemplateResolver">
    <property name="prefix" value="/WEB-INF/templates/"/>
    <property name="suffix" value=".html"/>
  </bean>
</beans>
```

###### 现代替代方案
你的 `**WebConfig**` 类通过以下方式替代了 SpringMVC.xml：

```xml
@Configuration
@ComponentScan("com.atguigu.mvc.controller")  // 替代 <context:component-scan>
@EnableWebMvc                                 // 替代 <mvc:annotation-driven>
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void configureDefaultServletHandling(...) {  // 替代 <mvc:default-servlet-handler>
        configurer.enable();
    }

    @Bean
    public CommonsMultipartResolver multipartResolver() {  // 替代 multipartResolver bean
        return new CommonsMultipartResolver();
    }

    // Thymeleaf 配置
    @Bean public ITemplateResolver templateResolver() { ... }
    @Bean public SpringTemplateEngine templateEngine() { ... }
    @Bean public ViewResolver viewResolver() { ... }
}
```

##### 3. 为什么这种新方式更好？
1. **类型安全**：Java 配置是类型安全的，编译器可以检查错误
2. **更灵活**：可以轻松添加条件逻辑和自定义行为
3. **更简洁**：减少了 XML 配置的冗余
4. **IDE 支持更好**：代码导航、重构和自动完成
5. **模块化**：可以轻松创建多个配置类并按需组合
6. **与 Spring Boot 兼容**：Spring Boot 完全基于 Java 配置

##### 4. 关键转换对照表

| **web.xml 功能** | **现代替代方案** |
| --- | --- |
| Servlet 定义 | `**AbstractAnnotationConfigDispatcherServletInitializer**` |
| Servlet 映射 | `**getServletMappings()**`<br/> 方法 |
| Filter 配置 | `**getServletFilters()**`<br/> 方法 |
| 上下文参数 | `**getRootConfigClasses()**`<br/>/`**getServletConfigClasses()**` |


| **SpringMVC.xml 功能** | **现代替代方案** |
| --- | --- |
| `**<mvc:annotation-driven>**` | `**@EnableWebMvc**` |
| `**<context:component-scan>**` | `**@ComponentScan**` |
| `**<mvc:default-servlet-handler>**` | `**configureDefaultServletHandling()**` |
| `**<bean>**`<br/> 定义 | `**@Bean**`<br/> 方法 |
| 拦截器配置 | `**addInterceptors()**`<br/> 方法 |


这种基于 Java 的配置方式是 Spring 3.0+ 推荐的做法，特别是在 Servlet 3.0+ 容器中运行时。

## 十三、SpringMVC执行流程
#### 1、SpringMVC常用组件
+ DispatcherServlet：**前端控制器**，不需要工程师开发，由框架提供

作用：统一处理请求和响应，整个流程控制的中心，由它调用其它组件处理用户的请求

+ HandlerMapping：**处理器映射器**，不需要工程师开发，由框架提供

作用：根据请求的url、method等信息查找Handler，即控制器方法

+ Handler：**处理器**，需要工程师开发

作用：在DispatcherServlet的控制下Handler对具体的用户请求进行处理

+ HandlerAdapter：**处理器适配器**，不需要工程师开发，由框架提供

作用：通过HandlerAdapter对处理器（控制器方法）进行执行

+ ViewResolver：**视图解析器**，不需要工程师开发，由框架提供

作用：进行视图解析，得到相应的视图，例如：ThymeleafView、InternalResourceView、RedirectView

+ View：**视图**

作用：将模型数据通过页面展示给用户

#### 2、DispatcherServlet初始化过程
DispatcherServlet 本质上是一个 Servlet，所以天然的遵循 Servlet 的生命周期。所以宏观上是 Servlet 生命周期来进行调度。

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1754593106573-794a65a4-5db4-4013-9ec5-c786211adcca.png)

###### a>初始化WebApplicationContext
所在类：org.springframework.web.servlet.FrameworkServlet

```xml
protected WebApplicationContext initWebApplicationContext() {
    WebApplicationContext rootContext =
        WebApplicationContextUtils.getWebApplicationContext(getServletContext());
    WebApplicationContext wac = null;

    if (this.webApplicationContext != null) {
        
        wac = this.webApplicationContext;
        if (wac instanceof ConfigurableWebApplicationContext) {
            ConfigurableWebApplicationContext cwac = (ConfigurableWebApplicationContext) wac;
            if (!cwac.isActive()) {
                
                
                if (cwac.getParent() == null) {
                    
                    
                    cwac.setParent(rootContext);
                }
                configureAndRefreshWebApplicationContext(cwac);
            }
        }
    }
    if (wac == null) {
        
        
        
        
        wac = findWebApplicationContext();
    }
    if (wac == null) {
        
        
        wac = createWebApplicationContext(rootContext);
    }

    if (!this.refreshEventReceived) {
        
        
        
        synchronized (this.onRefreshMonitor) {
            
            onRefresh(wac);
        }
    }

    if (this.publishContext) {
        
        
        String attrName = getServletContextAttributeName();
        getServletContext().setAttribute(attrName, wac);
    }

    return wac;
}
```

###### b>创建WebApplicationContext
所在类：org.springframework.web.servlet.FrameworkServlet

```xml
protected WebApplicationContext createWebApplicationContext(@Nullable ApplicationContext parent) {
    Class<?> contextClass = getContextClass();
    if (!ConfigurableWebApplicationContext.class.isAssignableFrom(contextClass)) {
        throw new ApplicationContextException(
            "Fatal initialization error in servlet with name '" + getServletName() +
            "': custom WebApplicationContext class [" + contextClass.getName() +
            "] is not of type ConfigurableWebApplicationContext");
    }
    
    ConfigurableWebApplicationContext wac =
        (ConfigurableWebApplicationContext) BeanUtils.instantiateClass(contextClass);

    wac.setEnvironment(getEnvironment());
    
    wac.setParent(parent);
    String configLocation = getContextConfigLocation();
    if (configLocation != null) {
        wac.setConfigLocation(configLocation);
    }
    configureAndRefreshWebApplicationContext(wac);

    return wac;
}
```

###### c>DispatcherServlet初始化策略
FrameworkServlet创建WebApplicationContext后，刷新容器，调用onRefresh(wac)，此方法在DispatcherServlet中进行了重写，调用了initStrategies(context)方法，初始化策略，即初始化DispatcherServlet的各个组件

所在类：org.springframework.web.servlet.DispatcherServlet

```xml
protected void initStrategies(ApplicationContext context) {
   initMultipartResolver(context);
   initLocaleResolver(context);
   initThemeResolver(context);
   initHandlerMappings(context);
   initHandlerAdapters(context);
   initHandlerExceptionResolvers(context);
   initRequestToViewNameTranslator(context);
   initViewResolvers(context);
   initFlashMapManager(context);
}
```

#### 3、DispatcherServlet调用组件处理请求
###### a>processRequest()
FrameworkServlet重写HttpServlet中的service()和doXxx()，这些方法中调用了processRequest(request, response)

所在类：org.springframework.web.servlet.FrameworkServlet

```xml
protected final void processRequest(HttpServletRequest request, HttpServletResponse response)
    throws ServletException, IOException {

    long startTime = System.currentTimeMillis();
    Throwable failureCause = null;

    LocaleContext previousLocaleContext = LocaleContextHolder.getLocaleContext();
    LocaleContext localeContext = buildLocaleContext(request);

    RequestAttributes previousAttributes = RequestContextHolder.getRequestAttributes();
    ServletRequestAttributes requestAttributes = buildRequestAttributes(request, response, previousAttributes);

    WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);
    asyncManager.registerCallableInterceptor(FrameworkServlet.class.getName(), new RequestBindingInterceptor());

    initContextHolders(request, localeContext, requestAttributes);

    try {
		
        doService(request, response);
    }
    catch (ServletException | IOException ex) {
        failureCause = ex;
        throw ex;
    }
    catch (Throwable ex) {
        failureCause = ex;
        throw new NestedServletException("Request processing failed", ex);
    }

    finally {
        resetContextHolders(request, previousLocaleContext, previousAttributes);
        if (requestAttributes != null) {
            requestAttributes.requestCompleted();
        }
        logResult(request, response, failureCause, asyncManager);
        publishRequestHandledEvent(request, response, startTime, failureCause);
    }
}
```

###### b>doService()
所在类：org.springframework.web.servlet.DispatcherServlet

```xml
@Override
protected void doService(HttpServletRequest request, HttpServletResponse response) throws Exception {
    logRequest(request);

    
    
    Map<String, Object> attributesSnapshot = null;
    if (WebUtils.isIncludeRequest(request)) {
        attributesSnapshot = new HashMap<>();
        Enumeration<?> attrNames = request.getAttributeNames();
        while (attrNames.hasMoreElements()) {
            String attrName = (String) attrNames.nextElement();
            if (this.cleanupAfterInclude || attrName.startsWith(DEFAULT_STRATEGIES_PREFIX)) {
                attributesSnapshot.put(attrName, request.getAttribute(attrName));
            }
        }
    }

    
    request.setAttribute(WEB_APPLICATION_CONTEXT_ATTRIBUTE, getWebApplicationContext());
    request.setAttribute(LOCALE_RESOLVER_ATTRIBUTE, this.localeResolver);
    request.setAttribute(THEME_RESOLVER_ATTRIBUTE, this.themeResolver);
    request.setAttribute(THEME_SOURCE_ATTRIBUTE, getThemeSource());

    if (this.flashMapManager != null) {
        FlashMap inputFlashMap = this.flashMapManager.retrieveAndUpdate(request, response);
        if (inputFlashMap != null) {
            request.setAttribute(INPUT_FLASH_MAP_ATTRIBUTE, Collections.unmodifiableMap(inputFlashMap));
        }
        request.setAttribute(OUTPUT_FLASH_MAP_ATTRIBUTE, new FlashMap());
        request.setAttribute(FLASH_MAP_MANAGER_ATTRIBUTE, this.flashMapManager);
    }

    RequestPath requestPath = null;
    if (this.parseRequestPath && !ServletRequestPathUtils.hasParsedRequestPath(request)) {
        requestPath = ServletRequestPathUtils.parseAndCache(request);
    }

    try {
        
        doDispatch(request, response);
    }
    finally {
        if (!WebAsyncUtils.getAsyncManager(request).isConcurrentHandlingStarted()) {
            
            if (attributesSnapshot != null) {
                restoreAttributesAfterInclude(request, attributesSnapshot);
            }
        }
        if (requestPath != null) {
            ServletRequestPathUtils.clearParsedRequestPath(request);
        }
    }
}
```

###### c>doDispatch()
所在类：org.springframework.web.servlet.DispatcherServlet

```xml
protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
    HttpServletRequest processedRequest = request;
    HandlerExecutionChain mappedHandler = null;
    boolean multipartRequestParsed = false;

    WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);

    try {
        ModelAndView mv = null;
        Exception dispatchException = null;

        try {
            processedRequest = checkMultipart(request);
            multipartRequestParsed = (processedRequest != request);

            
            
            mappedHandler = getHandler(processedRequest);
            if (mappedHandler == null) {
                noHandlerFound(processedRequest, response);
                return;
            }

            
           	
            HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());

            
            String method = request.getMethod();
            boolean isGet = "GET".equals(method);
            if (isGet || "HEAD".equals(method)) {
                long lastModified = ha.getLastModified(request, mappedHandler.getHandler());
                if (new ServletWebRequest(request, response).checkNotModified(lastModified) && isGet) {
                    return;
                }
            }
			
            
            if (!mappedHandler.applyPreHandle(processedRequest, response)) {
                return;
            }

            
            
            mv = ha.handle(processedRequest, response, mappedHandler.getHandler());

            if (asyncManager.isConcurrentHandlingStarted()) {
                return;
            }

            applyDefaultViewName(processedRequest, mv);
            
            mappedHandler.applyPostHandle(processedRequest, response, mv);
        }
        catch (Exception ex) {
            dispatchException = ex;
        }
        catch (Throwable err) {
            
            
            dispatchException = new NestedServletException("Handler dispatch failed", err);
        }
        
        processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException);
    }
    catch (Exception ex) {
        triggerAfterCompletion(processedRequest, response, mappedHandler, ex);
    }
    catch (Throwable err) {
        triggerAfterCompletion(processedRequest, response, mappedHandler,
                               new NestedServletException("Handler processing failed", err));
    }
    finally {
        if (asyncManager.isConcurrentHandlingStarted()) {
            
            if (mappedHandler != null) {
                mappedHandler.applyAfterConcurrentHandlingStarted(processedRequest, response);
            }
        }
        else {
            
            if (multipartRequestParsed) {
                cleanupMultipart(processedRequest);
            }
        }
    }
}
```

###### d>processDispatchResult()
```xml
private void processDispatchResult(HttpServletRequest request, HttpServletResponse response,
                                   @Nullable HandlerExecutionChain mappedHandler, @Nullable ModelAndView mv,
                                   @Nullable Exception exception) throws Exception {

    boolean errorView = false;

    if (exception != null) {
        if (exception instanceof ModelAndViewDefiningException) {
            logger.debug("ModelAndViewDefiningException encountered", exception);
            mv = ((ModelAndViewDefiningException) exception).getModelAndView();
        }
        else {
            Object handler = (mappedHandler != null ? mappedHandler.getHandler() : null);
            mv = processHandlerException(request, response, handler, exception);
            errorView = (mv != null);
        }
    }

    
    if (mv != null && !mv.wasCleared()) {
        
        render(mv, request, response);
        if (errorView) {
            WebUtils.clearErrorRequestAttributes(request);
        }
    }
    else {
        if (logger.isTraceEnabled()) {
            logger.trace("No view rendering, null ModelAndView returned.");
        }
    }

    if (WebAsyncUtils.getAsyncManager(request).isConcurrentHandlingStarted()) {
        
        return;
    }

    if (mappedHandler != null) {
        
        
        mappedHandler.triggerAfterCompletion(request, response, null);
    }
}
```

#### 4、SpringMVC的执行流程
1. 用户向服务器发送请求，请求被SpringMVC 前端控制器 DispatcherServlet捕获。
2. DispatcherServlet对请求URL进行解析，得到请求资源标识符（URI），判断请求URI对应的映射：

a) 不存在

i. 再判断是否配置了mvc:default-servlet-handler

ii. 如果没配置，则控制台报映射查找不到，客户端展示404错误

[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-QmMj41Xh-1651312070876)(img/img006.png)]

[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-uzwDWdQL-1651312070877)(img/img007.png)]

iii. 如果有配置，则访问目标资源（一般为静态资源，如：JS,CSS,HTML），找不到客户端也会展示404错误

[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-iZ8OfqD5-1651312070877)(img/img008.png)]

[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-KwFH1hXi-1651312070877)(img/img009.png)]

b) 存在则执行下面的流程

1. 根据该URI，调用HandlerMapping获得该Handler配置的所有相关的对象（包括Handler对象以及Handler对象对应的拦截器），最后以HandlerExecutionChain执行链对象的形式返回。
2. DispatcherServlet 根据获得的Handler，选择一个合适的HandlerAdapter。
3. 如果成功获得HandlerAdapter，此时将开始执行拦截器的preHandler(…)方法【正向】
4. 提取Request中的模型数据，填充Handler入参，开始执行Handler（Controller)方法，处理请求。在填充Handler的入参过程中，根据你的配置，Spring将帮你做一些额外的工作：

a) HttpMessageConveter： 将请求消息（如Json、xml等数据）转换成一个对象，将对象转换为指定的响应信息

b) 数据转换：对请求消息进行数据转换。如String转换成Integer、Double等

c) 数据格式化：对请求消息进行数据格式化。 如将字符串转换成格式化数字或格式化日期等

d) 数据验证： 验证数据的有效性（长度、格式等），验证结果存储到BindingResult或Error中

1. Handler执行完成后，向DispatcherServlet 返回一个ModelAndView对象。
2. 此时将开始执行拦截器的postHandle(…)方法【逆向】。
3. 根据返回的ModelAndView（此时会判断是否存在异常：如果存在异常，则执行HandlerExceptionResolver进行异常处理）选择一个适合的ViewResolver进行视图解析，根据Model和View，来渲染视图。
4. 渲染视图完毕执行拦截器的afterCompletion(…)方法【逆向】。

= null);  
  
}

```xml
// Did the handler return a view to render?
if (mv != null && !mv.wasCleared()) {
    // 处理模型数据和渲染视图
    render(mv, request, response);
    if (errorView) {
        WebUtils.clearErrorRequestAttributes(request);
    }
}
else {
    if (logger.isTraceEnabled()) {
        logger.trace("No view rendering, null ModelAndView returned.");
    }
}

if (WebAsyncUtils.getAsyncManager(request).isConcurrentHandlingStarted()) {
    // Concurrent handling started during a forward
    return;
}

if (mappedHandler != null) {
    // Exception (if any) is already handled..
    // 调用拦截器的afterCompletion()
    mappedHandler.triggerAfterCompletion(request, response, null);
}
```

}

```xml
### 4、SpringMVC的执行流程

1) 用户向服务器发送请求，请求被SpringMVC 前端控制器 DispatcherServlet捕获。

2) DispatcherServlet对请求URL进行解析，得到请求资源标识符（URI），判断请求URI对应的映射：

a) 不存在

i. 再判断是否配置了mvc:default-servlet-handler

ii. 如果没配置，则控制台报映射查找不到，客户端展示404错误

![在这里插入图片描述](https://img-blog.csdnimg.cn/79900b21d47f4574beb9219442894700.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/e7b37947fb83488282a9023153e8979a.png)


iii. 如果有配置，则访问目标资源（一般为静态资源，如：JS,CSS,HTML），找不到客户端也会展示404错误
![在这里插入图片描述](https://img-blog.csdnimg.cn/4e03ccf0d0dc4737a1a3f5d36c2c87e0.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/71a610163d5140e09af3d9d5661c3f3b.png)



b) 存在则执行下面的流程

3) 根据该URI，调用HandlerMapping获得该Handler配置的所有相关的对象（包括Handler对象以及Handler对象对应的拦截器），最后以HandlerExecutionChain执行链对象的形式返回。

4) DispatcherServlet 根据获得的Handler，选择一个合适的HandlerAdapter。

5) 如果成功获得HandlerAdapter，此时将开始执行拦截器的preHandler(…)方法【正向】

6) 提取Request中的模型数据，填充Handler入参，开始执行Handler（Controller)方法，处理请求。在填充Handler的入参过程中，根据你的配置，Spring将帮你做一些额外的工作：

a) HttpMessageConveter： 将请求消息（如Json、xml等数据）转换成一个对象，将对象转换为指定的响应信息

b) 数据转换：对请求消息进行数据转换。如String转换成Integer、Double等

c) 数据格式化：对请求消息进行数据格式化。 如将字符串转换成格式化数字或格式化日期等

d) 数据验证： 验证数据的有效性（长度、格式等），验证结果存储到BindingResult或Error中

7) Handler执行完成后，向DispatcherServlet 返回一个ModelAndView对象。

8) 此时将开始执行拦截器的postHandle(...)方法【逆向】。

9) 根据返回的ModelAndView（此时会判断是否存在异常：如果存在异常，则执行HandlerExceptionResolver进行异常处理）选择一个适合的ViewResolver进行视图解析，根据Model和View，来渲染视图。

10) 渲染视图完毕执行拦截器的afterCompletion(…)方法【逆向】。

11) 将渲染结果返回给客户端。
```





