## 2 关于Cloud各种组件的停更/升级/替换
### 2.1 微服务零基础理论知识入门
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756886931249-4297b845-694a-4fb4-925f-70e242e2dc69.png)

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756886931153-c897790c-c31e-44a1-a0a6-3da19aea2723.png)

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756886931575-820df259-3ef5-4934-bff3-c37bb89f1d75.png)

### 2.2 SpringCloud是什么？能干吗？产生背景？
让程序员专注于业务逻辑，有第3方支援

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756886931558-9ba508ab-90c1-4a83-8e58-99ceda988f5e.png)

### 2.3 本次讲解定稿，速通版
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756886931216-18edaf36-94fa-490f-8034-c8caf58999b6.png)

**Netflix OSS被移除的原因**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756886931716-91e875b6-f7e2-430c-8114-723fcd471a2a.png)

**Netflix哪些被移除了？**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756886931922-43140e8d-fa07-41d5-909e-80a84ea5fd12.png)

**Spring Cloud Netflix项目进入维护模式**

停更不停用

被动修复bugs；不再接受合并请求；不再发布新版本

**由停更引发的“升级惨案”**

明细条目

2020第二季

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756886932004-3698eabc-657d-4c37-a6b2-8b1c5e833b3a.png)

2024第三季

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756886932340-d608b071-6d42-45c5-9e92-e9cda0a595a5.png)

备注，如果被remove掉的组件，不再使用

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756886932576-5343a1af-1f20-44ea-9def-8e51d0ec31f5.png)

## 3 微服务架构编码Base工程模块构建
### 3.1 订单→支付，业务需求说明
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756886954382-fa0f1f27-0a63-43d4-a1af-f2400d5c5ca9.png)

### 3.2 约定 > 配置 > 编码
### 3.3 IDEA新建Project和Maven父工程
#### 3.3.1 微服务cloud整体聚合Maven父工程Project
**Maven父工程步骤**

**1 New Project**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756886954520-edb850fb-5adf-49ec-a9e8-d3c03d08b69e.png)

**2 聚合总父工程名字**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756886955088-15391481-8cb3-498c-9789-bd38e2c5d2fd.png)

**3 字符编码**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756886954513-8cf72989-24fc-4b05-b922-bc2e462494ad.png)

**4 注解生效激活**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756886954638-3b4ceb24-244a-4a6b-9ab1-01ac25bde68f.png)

**5 java编译版本选17**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756886955199-2360d5e4-69cc-40a8-8457-e6bf53c274cd.png)

**6 File Type过滤**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756886955217-fd81f1d3-f6c6-4029-bad4-3d47014e767b.png)

#### 3.3.2 父工程POM文件内容
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.atguigu.cloud</groupId>
  <artifactId>mscloudV5</artifactId>
  <version>1.0-SNAPSHOT</version>
  <packaging>pom</packaging>

  <properties>
    <maven.compiler.source>17</maven.compiler.source>
    <maven.compiler.target>17</maven.compiler.target>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <hutool.version>5.8.22</hutool.version>
    <lombok.version>1.18.26</lombok.version>
    <druid.version>1.1.20</druid.version>
    <mybatis.springboot.version>3.0.2</mybatis.springboot.version>
    <mysql.version>8.0.11</mysql.version>
    <swagger3.version>2.2.0</swagger3.version>
    <mapper.version>4.2.3</mapper.version>
    <fastjson2.version>2.0.40</fastjson2.version>
    <persistence-api.version>1.0.2</persistence-api.version>
    <spring.boot.test.version>3.1.5</spring.boot.test.version>
    <spring.boot.version>3.2.0</spring.boot.version>
    <spring.cloud.version>2023.0.0</spring.cloud.version>
    <spring.cloud.alibaba.version>2022.0.0.0-RC2</spring.cloud.alibaba.version>
  </properties>

  <dependencyManagement>
    <dependencies>
      <!--springboot 3.2.0-->
      <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>${spring.boot.version}</version>
        <type>pom</type>
        <scope>import</scope>
      </dependency>
      <!--springcloud 2023.0.0-->
      <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-dependencies</artifactId>
        <version>${spring.cloud.version}</version>
        <type>pom</type>
        <scope>import</scope>
      </dependency>
      <!--springcloud alibaba 2022.0.0.0-RC2-->
      <dependency>
        <groupId>com.alibaba.cloud</groupId>
        <artifactId>spring-cloud-alibaba-dependencies</artifactId>
        <version>${spring.cloud.alibaba.version}</version>
        <type>pom</type>
        <scope>import</scope>
      </dependency>
      <!--SpringBoot集成mybatis-->
      <dependency>
        <groupId>org.mybatis.spring.boot</groupId>
        <artifactId>mybatis-spring-boot-starter</artifactId>
        <version>${mybatis.springboot.version}</version>
      </dependency>
      <!--Mysql数据库驱动8 -->
      <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>${mysql.version}</version>
            </dependency>
            <!--SpringBoot集成druid连接池-->
            <dependency>
                <groupId>com.alibaba</groupId>
                <artifactId>druid-spring-boot-starter</artifactId>
                <version>${druid.version}</version>
            </dependency>
            <!--通用Mapper4之tk.mybatis-->
            <dependency>
                <groupId>tk.mybatis</groupId>
                <artifactId>mapper</artifactId>
                <version>${mapper.version}</version>
            </dependency>
            <!--persistence-->
            <dependency>
                <groupId>javax.persistence</groupId>
                <artifactId>persistence-api</artifactId>
                <version>${persistence-api.version}</version>
            </dependency>
            <!-- fastjson2 -->
            <dependency>
                <groupId>com.alibaba.fastjson2</groupId>
                <artifactId>fastjson2</artifactId>
                <version>${fastjson2.version}</version>
            </dependency>
            <!-- swagger3 调用方式 http://你的主机IP地址:5555/swagger-ui/index.html -->
            <dependency>
                <groupId>org.springdoc</groupId>
                <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
                <version>${swagger3.version}</version>
            </dependency>
            <!--hutool-->
            <dependency>
                <groupId>cn.hutool</groupId>
                <artifactId>hutool-all</artifactId>
                <version>${hutool.version}</version>
            </dependency>
            <!--lombok-->
            <dependency>
                <groupId>org.projectlombok</groupId>
                <artifactId>lombok</artifactId>
                <version>${lombok.version}</version>
                <optional>true</optional>
            </dependency>
            <!-- spring-boot-starter-test -->
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-test</artifactId>
                <version>${spring.boot.test.version}</version>
                <scope>test</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
</project>
```

#### 3.3.3 Maven工程落地细节复习
**Maven中的DependencyManagement和Dependencies**

`dependencyManagement`

Maven 使用dependencyManagement 元素来提供了一种管理依赖版本号的方式。通常会在一个组织或者项目的最顶层的父POM 中看到dependencyManagement 元素。使用pom.xml 中的dependencyManagement 元素能让所有在子项目中引用一个依赖而不用显式的列出版本号。Maven会沿着父子层次向上走，直到找到一个拥有dependencyManagement 元素的项目，然后它就会使用这个dependencyManagement 元素中指定的版本号。

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756886955291-317e9747-b496-475f-a3da-b19e782908b8.png)

这样做的好处就是：如果有多个子项目都引用同一样依赖，则可以避免在每个使用的子项目里都声明一个版本号，优势：

| 1 | 这样当想升级或切换到另一个版本时，只需要在顶层父容器里更新，而不需要一个一个子项目的修改 ； |
| :--- | :--- |
| 2 | 另外如果某个子项目需要另外的一个版本，只需要声明version就可。 |


*     dependencyManagement里只是声明依赖，_**并不实现引入**_，因此子项目需要显式的声明需要用的依赖。

*   如果不在子项目中声明依赖，是不会从父项目中继承下来的，只有在子项目中写了该依赖项并且没有指定具体版本，才会从父项目中继承该项        且version和scope都读取自父pom;

*     如果子项目中指定了版本号，那么会使用子项目中指定的jar版本。

**maven中跳过单元测试**

1  配置

| < **build** > _<!-- maven_ _中跳过单元测试 _ _-->   _ < **plugins** >    < **plugin** >    < **groupId** >org.apache.maven.plugins </ **groupId** >    < **artifactId** >maven-surefire-plugin </ **artifactId** >    < **configuration** >    < **skip** >true </ **skip** >    </ **configuration** >    </ **plugin** >    </ **plugins** >   </ **build** > |
| :--- |


2  IDEA工具支持(推荐)

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756886955340-beee26a9-7499-454b-aa0d-f775bd9ce3f7.png)

#### 3.3.4 mysql驱动说明
**Mysql5**

```plain
便笺
# mysql5.7---JDBC四件套
jdbc.driverClass = com.mysql.jdbc.Driver
jdbc.url= jdbc:mysql://localhost:3306/db2024?useUnicode=true&characterEncoding=UTF-8&useSSL=false
jdbc.user = root
jdbc.password =123456
 


# Maven的POM文件处理
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>5.1.47</version>
</dependency>
```

**Mysql8**

```plain
# mysql8.0---JDBC四件套
jdbc.driverClass = com.mysql.cj.jdbc.Driver
jdbc.url= jdbc:mysql://localhost:3306/db2024?characterEncoding=utf8&useSSL=false&serverTimezone=GMT%2B8&rewriteBatchedStatements=true&allowPublicKeyRetrieval=true
jdbc.user = root
jdbc.password =123456

 


# Maven的POM
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>8.0.11</version>
</dependency>
```

### 3.4 Mapper4之一键生成
**mybatis-generator**

[MyBatis Generator Core – Introduction to MyBatis Generator](http://mybatis.org/generator/)

**MyBatis通用Mapper4官网**

[https://github.com/abel533/Mapper](https://github.com/abel533/Mapper)

本次使用Mapper4

**下一代：MyBatis 通用 Mapper5官网**

[https://github.com/mybatis-mapper/mapper](https://github.com/mybatis-mapper/mapper)

#### **3.4.1 一键生成步骤**
**SQL**

db2024库t_pay支付信息表SQL

```plsql
DROP TABLE IF EXISTS `t_pay`;

 
CREATE TABLE `t_pay` (

  `id` INT(10) UNSIGNED NOT NULL AUTO_INCREMENT,

  `pay_no` VARCHAR(50) NOT NULL COMMENT '支付流水号',

  `order_no` VARCHAR(50) NOT NULL COMMENT '订单流水号',

  `user_id` INT(10) DEFAULT '1' COMMENT '用户账号ID',

  `amount` DECIMAL(8,2) NOT NULL DEFAULT '9.9' COMMENT '交易金额',

  `deleted` TINYINT(4) UNSIGNED NOT NULL DEFAULT '0' COMMENT '删除标志，默认0不删除，1删除',

  `create_time` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',

  `update_time` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
  PRIMARY KEY (`id`)
) ENGINE=INNODB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8mb4 COMMENT='支付交易表';

 

INSERT INTO t_pay(pay_no,order_no) VALUES('pay17203699','6544bafb424a');


SELECT * FROM t_pay;
```

**Module**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756886956085-ac5bed3c-95bf-4f0d-9ef0-d7e1ad78fc87.png)

普通Maven工程

mybatis_generator2024

**POM**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <parent>
    <groupId>com.atguigu.cloud</groupId>
    <artifactId>mscloudV5</artifactId>
    <version>1.0-SNAPSHOT</version>
  </parent>

  <!--我自己独一份，只是一个普通Maven工程，与boot和cloud无关-->
  <artifactId>mybatis_generator2024</artifactId>

  <properties>
    <maven.compiler.source>17</maven.compiler.source>
    <maven.compiler.target>17</maven.compiler.target>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
  </properties>

  <dependencies>
    <!--Mybatis 通用mapper tk单独使用，自己独有+自带版本号-->
    <dependency>
      <groupId>org.mybatis</groupId>
      <artifactId>mybatis</artifactId>
      <version>3.5.13</version>
    </dependency>
    <!-- Mybatis Generator 自己独有+自带版本号-->
    <dependency>
      <groupId>org.mybatis.generator</groupId>
      <artifactId>mybatis-generator-core</artifactId>
      <version>1.4.2</version>
    </dependency>
    <!--通用Mapper-->
    <dependency>
      <groupId>tk.mybatis</groupId>
      <artifactId>mapper</artifactId>
    </dependency>
    <!--mysql8.0-->
    <dependency>
      <groupId>mysql</groupId>
      <artifactId>mysql-connector-java</artifactId>
    </dependency>
    <!--persistence-->
    <dependency>
      <groupId>javax.persistence</groupId>
      <artifactId>persistence-api</artifactId>
    </dependency>
    <!--hutool-->
    <dependency>
      <groupId>cn.hutool</groupId>
      <artifactId>hutool-all</artifactId>
    </dependency>
    <!--lombok-->
    <dependency>
      <groupId>org.projectlombok</groupId>
      <artifactId>lombok</artifactId>
      <optional>true</optional>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-test</artifactId>
      <scope>test</scope>
      <exclusions>
        <exclusion>
          <groupId>org.junit.vintage</groupId>
          <artifactId>junit-vintage-engine</artifactId>
        </exclusion>
      </exclusions>
    </dependency>
  </dependencies>

  <build>
    <resources>
      <resource>
        <directory>${basedir}/src/main/java</directory>
        <includes>
          <include>**/*.xml</include>
        </includes>
      </resource>
      <resource>
        <directory>${basedir}/src/main/resources</directory>
      </resource>
    </resources>
    <plugins>
      <plugin>
        <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <excludes>
                        <exclude>
                            <groupId>org.projectlombok</groupId>
                            <artifactId>lombok</artifactId>
                        </exclude>
                    </excludes>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.mybatis.generator</groupId>
                <artifactId>mybatis-generator-maven-plugin</artifactId>
                <version>1.4.2</version>
                <configuration>
                    <configurationFile>${basedir}/src/main/resources/generatorConfig.xml</configurationFile>
                    <overwrite>true</overwrite>
                    <verbose>true</verbose>
                </configuration>
                <dependencies>
                    <dependency>
                        <groupId>mysql</groupId>
                        <artifactId>mysql-connector-java</artifactId>
                        <version>8.0.33</version>
                    </dependency>
                    <dependency>
                        <groupId>tk.mybatis</groupId>
                        <artifactId>mapper</artifactId>
                        <version>4.2.3</version>
                    </dependency>
                </dependencies>
            </plugin>
        </plugins>
    </build>

</project>
```

**配置**

**src\main\resources路径下新建**

config.properties

Mysql5

```properties
#User表包名
package.name=com.atguigu.cloud
# mysql5.7
jdbc.driverClass = com.mysql.jdbc.Driver
jdbc.url= jdbc:mysql://localhost:3306/db2024?useUnicode=true&characterEncoding=UTF-8&useSSL=false
jdbc.user = root
jdbc.password =123456
```

Mysql8

```properties
#t_pay表包名
package.name=com.atguigu.cloud

# mysql8.0
jdbc.driverClass = com.mysql.cj.jdbc.Driver
jdbc.url= jdbc:mysql://localhost:3306/db2024?characterEncoding=utf8&useSSL=false&serverTimezone=GMT%2B8&rewriteBatchedStatements=true&allowPublicKeyRetrieval=true
jdbc.user = root
jdbc.password =123456
```

generatorConfig.xml

内容

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
"http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">

<generatorConfiguration>
  <properties resource="config.properties"/>

  <context id="Mysql" targetRuntime="MyBatis3Simple" defaultModelType="flat">
    <property name="beginningDelimiter" value="`"/>
    <property name="endingDelimiter" value="`"/>

    <plugin type="tk.mybatis.mapper.generator.MapperPlugin">
      <property name="mappers" value="tk.mybatis.mapper.common.Mapper"/>
      <property name="caseSensitive" value="true"/>
    </plugin>

    <jdbcConnection driverClass="${jdbc.driverClass}"
      connectionURL="${jdbc.url}"
      userId="${jdbc.user}"
      password="${jdbc.password}">
    </jdbcConnection>

    <javaModelGenerator targetPackage="${package.name}.entities" targetProject="src/main/java"/>

    <sqlMapGenerator targetPackage="${package.name}.mapper" targetProject="src/main/java"/>

    <javaClientGenerator targetPackage="${package.name}.mapper" targetProject="src/main/java" type="XMLMAPPER"/>

    <table tableName="t_pay" domainObjectName="Pay">
      <generatedKey column="id" sqlStatement="JDBC"/>
    </table>
  </context>
</generatorConfiguration>
```

**一键生成**

双击插件mybatis-generator:gererate，一键生成entity+mapper接口+xml实现SQL

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756886956525-b6861662-a7a3-4595-b552-8c59797534dc.png)

### 3.5 Rest通用Base工程构建
#### 3.5.1 工程V1
##### 3.5.1.1 cloud-provider-payment8001 微服务提供者支付Module模块
**微服务小口诀**

1 建module

2 改POM

3 写YML

4 主启动

5 业务类

**步骤**

**建module**

建普通Maven模块 cloud-provider-payment8001

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756886956282-7214f721-835e-45b7-b4d6-e027721cf3a2.png)

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756886956416-9b0934d2-e597-443a-a163-6f06a1f30aa0.png)

创建完成后请回到父工程查看pom文件变化

**改POM**

```plain
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>com.atguigu.cloud</groupId>
        <artifactId>mscloudV5</artifactId>
        <version>1.0-SNAPSHOT</version>
    </parent>

    <artifactId>cloud-provider-payment8001</artifactId>

    <properties>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>


    <dependencies>
        <!--SpringBoot通用依赖模块-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <!--SpringBoot集成druid连接池-->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid-spring-boot-starter</artifactId>
        </dependency>
        <!-- Swagger3 调用方式 http://你的主机IP地址:5555/swagger-ui/index.html -->
        <dependency>
            <groupId>org.springdoc</groupId>
            <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
        </dependency>
        <!--mybatis和springboot整合-->
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
        </dependency>
        <!--Mysql数据库驱动8 -->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
        </dependency>
        <!--persistence-->
        <dependency>
            <groupId>javax.persistence</groupId>
            <artifactId>persistence-api</artifactId>
        </dependency>
        <!--通用Mapper4-->
        <dependency>
            <groupId>tk.mybatis</groupId>
            <artifactId>mapper</artifactId>
        </dependency>
        <!--hutool-->
        <dependency>
            <groupId>cn.hutool</groupId>
            <artifactId>hutool-all</artifactId>
        </dependency>
        <!-- fastjson2 -->
        <dependency>
            <groupId>com.alibaba.fastjson2</groupId>
            <artifactId>fastjson2</artifactId>
        </dependency>
        <!--lombok-->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.18.28</version>
            <scope>provided</scope>
        </dependency>
        <!--test-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
```

**写YML**

```plain
server:
  port: 8001

# ==========applicationName + druid-mysql8 driver===================
spring:
  application:
    name: cloud-payment-service

  datasource:
    type: com.alibaba.druid.pool.DruidDataSource
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/db2024?characterEncoding=utf8&useSSL=false&serverTimezone=GMT%2B8&rewriteBatchedStatements=true&allowPublicKeyRetrieval=true
    username: root
    password: 123456

# ========================mybatis===================
mybatis:
  mapper-locations: classpath:mapper/*.xml
  type-aliases-package: com.atguigu.cloud.entities
  configuration:
    map-underscore-to-camel-case: true
```

**主启动（修改Main类名为Main8001）**

```plain
package com.atguigu.cloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import tk.mybatis.spring.annotation.MapperScan;

/**
 * @auther zzyy
 * @create 2023-11-03 17:54
 */
@SpringBootApplication
@MapperScan("com.atguigu.cloud.mapper") //import tk.mybatis.spring.annotation.MapperScan;
public class Main8001
{
    public static void main(String[] args)
    {
        SpringApplication.run(Main8001.class,args);
    }
}
```

**业务类**

将之前一键生成的代码直接拷贝进8001模块

**entities**

**主实体Pay**

```plain
package com.atguigu.cloud.entities;

import java.math.BigDecimal;
import java.util.Date;
import javax.persistence.*;

/**
 * 表名：t_pay
 * 表注释：支付交易表
*/
@Table(name = "t_pay")
public class Pay {
    @Id
    @GeneratedValue(generator = "JDBC")
    private Integer id;

    /**
     * 支付流水号
     */
    @Column(name = "pay_no")
    private String payNo;

    /**
     * 订单流水号
     */
    @Column(name = "order_no")
    private String orderNo;

    /**
     * 用户账号ID
     */
    @Column(name = "user_id")
    private Integer userId;

    /**
     * 交易金额
     */
    private BigDecimal amount;

    /**
     * 删除标志，默认0不删除，1删除
     */
    private Byte deleted;

    /**
     * 创建时间
     */
    @Column(name = "create_time")
    private Date createTime;

    /**
     * 更新时间
     */
    @Column(name = "update_time")
    private Date updateTime;

    /**
     * @return id
     */
    public Integer getId() {
        return id;
    }

    /**
     * @param id
     */
    public void setId(Integer id) {
        this.id = id;
    }

    /**
     * 获取支付流水号
     *
     * @return payNo - 支付流水号
     */
    public String getPayNo() {
        return payNo;
    }

    /**
     * 设置支付流水号
     *
     * @param payNo 支付流水号
     */
    public void setPayNo(String payNo) {
        this.payNo = payNo;
    }

    /**
     * 获取订单流水号
     *
     * @return orderNo - 订单流水号
     */
    public String getOrderNo() {
        return orderNo;
    }

    /**
     * 设置订单流水号
     *
     * @param orderNo 订单流水号
     */
    public void setOrderNo(String orderNo) {
        this.orderNo = orderNo;
    }

    /**
     * 获取用户账号ID
     *
     * @return userId - 用户账号ID
     */
    public Integer getUserId() {
        return userId;
    }

    /**
     * 设置用户账号ID
     *
     * @param userId 用户账号ID
     */
    public void setUserId(Integer userId) {
        this.userId = userId;
    }

    /**
     * 获取交易金额
     *
     * @return amount - 交易金额
     */
    public BigDecimal getAmount() {
        return amount;
    }

    /**
     * 设置交易金额
     *
     * @param amount 交易金额
     */
    public void setAmount(BigDecimal amount) {
        this.amount = amount;
    }

    /**
     * 获取删除标志，默认0不删除，1删除
     *
     * @return deleted - 删除标志，默认0不删除，1删除
     */
    public Byte getDeleted() {
        return deleted;
    }

    /**
     * 设置删除标志，默认0不删除，1删除
     *
     * @param deleted 删除标志，默认0不删除，1删除
     */
    public void setDeleted(Byte deleted) {
        this.deleted = deleted;
    }

    /**
     * 获取创建时间
     *
     * @return createTime - 创建时间
     */
    public Date getCreateTime() {
        return createTime;
    }

    /**
     * 设置创建时间
     *
     * @param createTime 创建时间
     */
    public void setCreateTime(Date createTime) {
        this.createTime = createTime;
    }

    /**
     * 获取更新时间
     *
     * @return updateTime - 更新时间
     */
    public Date getUpdateTime() {
        return updateTime;
    }

    /**
     * 设置更新时间
     *
     * @param updateTime 更新时间
     */
    public void setUpdateTime(Date updateTime) {
        this.updateTime = updateTime;
    }
}
```

**传递数值PayDTO**

```plain
package com.atguigu.cloud.entities;

import io.swagger.v3.oas.annotations.media.Schema;
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

import java.io.Serializable;
import java.math.BigDecimal;

/**
 * @auther zzyy
 * @create 2023-11-03 18:58
 */
@Data
@AllArgsConstructor
@NoArgsConstructor
public class PayDTO implements Serializable
{
    private Integer id;
    //支付流水号
    private String payNo;
    //订单流水号
    private String orderNo;
    //用户账号ID
    private Integer userId;
    //交易金额
    private BigDecimal amount;
}
```

**mapper**

**Mapper接口PayMapper**

```plain
package com.atguigu.cloud.mapper;

import com.atguigu.cloud.entities.Pay;
import tk.mybatis.mapper.common.Mapper;

public interface PayMapper extends Mapper<Pay> {
}
```

**映射文件PayMapper.xml**

src\main\resources路径下，新建文件夹mapper。拷贝PayMapper.xml进上一步的mapper文件夹

PayMapper.xml

```plain
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.atguigu.cloud.mapper.PayMapper">
  <resultMap id="BaseResultMap" type="com.atguigu.cloud.entities.Pay">
    <!--
      WARNING - @mbg.generated
    -->
    <id column="id" jdbcType="INTEGER" property="id" />
    <result column="pay_no" jdbcType="VARCHAR" property="payNo" />
    <result column="order_no" jdbcType="VARCHAR" property="orderNo" />
    <result column="user_id" jdbcType="INTEGER" property="userId" />
    <result column="amount" jdbcType="DECIMAL" property="amount" />
    <result column="deleted" jdbcType="TINYINT" property="deleted" />
    <result column="create_time" jdbcType="TIMESTAMP" property="createTime" />
    <result column="update_time" jdbcType="TIMESTAMP" property="updateTime" />
  </resultMap>
</mapper>
```

**service**

**服务接口PayService**

```plain
package com.atguigu.cloud.service;

import com.atguigu.cloud.entities.Pay;

import java.util.List;

public interface PayService
{
    public int add(Pay pay);
    public int delete(Integer id);
    public int update(Pay pay);
    
    public Pay   getById(Integer id);
    public List<Pay> getAll();
}
```

**实现类PayServiceImpl**

```plain
package com.atguigu.cloud.service.impl;

import com.atguigu.cloud.entities.Pay;
import com.atguigu.cloud.mapper.PayMapper;
import com.atguigu.cloud.service.PayService;
import jakarta.annotation.Resource;
import org.springframework.stereotype.Service;

import java.util.List;

/**
 * @auther zzyy
 * @create 2023-11-03 18:44
 */
@Service
public class PayServiceImpl implements PayService{
    @Resource
    PayMapper payMapper;
    @Override
    public int add(Pay pay){
        return payMapper.insertSelective(pay);
    }
    @Override
    public int delete(Integer id){
        return payMapper.deleteByPrimaryKey(id);
    }
    @Override
    public int update(Pay pay){
        return payMapper.updateByPrimaryKeySelective(pay);
    }
    @Override
    public Pay getById(Integer id){
        return payMapper.selectByPrimaryKey(id);
    }
    @Override
    public List<Pay> getAll(){
        return payMapper.selectAll();
    }
}
```

**controller**

```plain
package com.atguigu.cloud.controller;

import com.atguigu.cloud.entities.Pay;
import com.atguigu.cloud.entities.PayDTO;
import com.atguigu.cloud.service.PayService;
import io.swagger.v3.oas.annotations.Operation;
import io.swagger.v3.oas.annotations.tags.Tag;
import jakarta.annotation.Resource;
import org.springframework.beans.BeanUtils;
import org.springframework.web.bind.annotation.*;

/**
 * @auther zzyy
 * @create 2023-11-03 18:55
 */
@RestController
public class PayController{
    @Resource PayService payService;
    @PostMapping(value = "/pay/add")
    public String addPay(@RequestBody Pay pay){
        System.out.println(pay.toString());
        int i = payService.add(pay);
        return "成功插入记录，返回值："+i;
    }
    @DeleteMapping(value = "/pay/del/{id}")
    public Integer deletePay(@PathVariable("id") Integer id) {
        return payService.delete(id);
    }
    @PutMapping(value = "/pay/update")
    public String updatePay(@RequestBody PayDTO payDTO){
        Pay pay = new Pay();
        BeanUtils.copyProperties(payDTO, pay);

        int i = payService.update(pay);
        return "成功修改记录，返回值："+i;
    }
    @GetMapping(value = "/pay/get/{id}")
    public Pay getById(@PathVariable("id") Integer id){
        return payService.getById(id);
    }//全部查询getall作为家庭作业
}
```

**测试**

**PostMan**

**add**

| {<br/>"payNo": "17204076",<br/>"orderNo": "6544de1c424a",<br/>"userId": "2",<br/>"amount": "19.90"<br/>} |
| :--- |


<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756886983227-20e5a519-4cf3-4d50-adf0-0b0f32d70756.png)

json测试字段和我们的entity实体类字段一一对应

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756886983251-9108377b-4b83-4ce5-9b72-7c1a1cf3411b.png)

**delete**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756886983101-37d843c5-c9f4-4fbb-acf0-f9a82987365f.png)

**update**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756886983099-f8584c83-2d14-4af1-afe7-6ef84c5f6525.png)

**select**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756886983235-8e7cbf5e-1ab9-4723-976e-ea5d6964ff19.png)

**Swagger3**

**常用注解**

**注解列表**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887029222-d7e43527-5427-451d-bc42-26677c35f5ef.png)

**Controller**

@Tag

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887029062-8f654261-c7b9-4f82-b14f-22cdf230f242.png)

修改PayController

```plain
package com.atguigu.cloud.controller;

import com.atguigu.cloud.entities.Pay;
import com.atguigu.cloud.entities.PayDTO;
import com.atguigu.cloud.service.PayService;
import io.swagger.v3.oas.annotations.Operation;
import io.swagger.v3.oas.annotations.tags.Tag;
import jakarta.annotation.Resource;
import org.springframework.beans.BeanUtils;
import org.springframework.web.bind.annotation.*;

/**
 * @auther zzyy
 * @create 2023-11-03 18:55
 */
@RestController
@Tag(name = "支付微服务模块",description = "支付CRUD")
public class PayController
{
    @Resource
    PayService payService;

    @PostMapping(value = "/pay/add")
    @Operation(summary = "新增",description = "新增支付流水方法,json串做参数")
    public String addPay(@RequestBody Pay pay)
    {
        System.out.println(pay.toString());
        int i = payService.add(pay);
        return "成功插入记录，返回值："+i;
    }

    @DeleteMapping(value = "/pay/del/{id}")
    @Operation(summary = "删除",description = "删除支付流水方法")
    public Integer deletePay(@PathVariable("id") Integer id) {
        return payService.delete(id);
    }

    @PutMapping(value = "/pay/update")
    @Operation(summary = "修改",description = "修改支付流水方法")
    public String updatePay(@RequestBody PayDTO payDTO)
    {
        Pay pay = new Pay();
        BeanUtils.copyProperties(payDTO, pay);

        int i = payService.update(pay);
        return "成功修改记录，返回值："+i;
    }

    @GetMapping(value = "/pay/get/{id}")
    @Operation(summary = "按照ID查流水",description = "查询支付流水方法")
    public Pay getById(@PathVariable("id") Integer id)
    {
        return payService.getById(id);
    }

    //全部查询getall作为家庭作业
}
```

**方法**

@Operation

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887029096-78e9adff-9bce-4806-967b-d07a254e773d.png)

**entity或者DTO**

@Schema

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887029045-973a0404-fd95-4dbc-ac64-e55e513f4a1f.png)

**含分组迭代的Config配置类**

Swagger3Config

```plain
package com.atguigu.cloud.config;

import io.swagger.v3.oas.models.ExternalDocumentation;
import io.swagger.v3.oas.models.OpenAPI;
import io.swagger.v3.oas.models.info.Info;
import org.springdoc.core.models.GroupedOpenApi;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * @auther zzyy
 * @create 2023-11-04 10:40
 */
@Configuration
public class Swagger3Config
{
    @Bean
    public GroupedOpenApi PayApi()
    {
        return GroupedOpenApi.builder().group("支付微服务模块").pathsToMatch("/pay/**").build();
    }
    @Bean
    public GroupedOpenApi OtherApi()
    {
        return GroupedOpenApi.builder().group("其它微服务模块").pathsToMatch("/other/**", "/others").build();
    }
    /*@Bean
    public GroupedOpenApi CustomerApi()
    {
        return GroupedOpenApi.builder().group("客户微服务模块").pathsToMatch("/customer/**", "/customers").build();
    }*/

    @Bean
    public OpenAPI docsOpenApi()
    {
        return new OpenAPI()
                .info(new Info().title("cloud2024")
                        .description("通用设计rest")
                        .version("v1.0"))
                .externalDocs(new ExternalDocumentation()
                        .description("www.atguigu.com")
                        .url("https://yiyan.baidu.com/"));
    }
}
```

**调用方式**

[http://localhost:8001/swagger-ui/index.html](http://localhost:8001/swagger-ui/index.html)

##### 3.5.1.2 上述模块还有那些问题
**1 时间格式问题**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887028952-7dc6d7b4-77ab-4239-b0b2-7a2e66151477.png)

时间日志格式的统一和定制情况？

**2 Java如何设计API接口实现统一格式返回？**

**影响前端/小程序/app等交互体验和开发**

void

数值

String

对象entity

Map

......

**看看目前程序返回值情况**

故意写了多种返回类值

**3 全局异常接入返回的标准格式**

有统一返回值+全局统一异常

#### 3.5.2 工程V2
cloud-provider-payment8001 微服务提供者支付Module模块V2改进版++

##### 3.5.2.1 解决：时间格式问题
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887029955-10cb8d60-9b94-4b1a-aa1d-116c79c3ea55.png)

|    _/**   __* __创建时间   __*/_   @Column(name = **"create_time"**)   @JsonFormat(pattern = **"yyyy-MM-dd HH:mm:ss"**, timezone = **"GMT+8"**)   **private **Date **createTime**;      _/**   __* __更新时间   __*/_   @Column(name = **"update_time"**)   @JsonFormat(pattern = **"yyyy-MM-dd HH:mm:ss"**, timezone = **"GMT+8"**)   **private **Date **updateTime**; |
| :--- |


##### 3.5.2.2 解决：统一返回值
**思路**

**定义返回标准格式，3大标配**

code状态值：由后端统一定义各种返回结果的状态码

message描述：本次接口调用的结果描述

data数据：本次返回的数据

**扩展**

接口调用时间之类

timestamp: 接口调用时间

**步骤**

**新建枚举类ReturnCodeEnum**

HTTP请求返回的状态码

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887030983-00144891-0901-4b56-9b68-b1fef332f3dc.png)

ReturnCodeEnum

```plain
package com.atguigu.cloud.resp;

import lombok.Getter;

import java.util.Arrays;

/**
 * @auther zzyy
 * @create 2023-11-04 11:51
 */
@Getter
public enum ReturnCodeEnum
{
    /**操作失败**/
    RC999("999","操作XXX失败"),
    /**操作成功**/
    RC200("200","success"),
    /**服务降级**/
    RC201("201","服务开启降级保护,请稍后再试!"),
    /**热点参数限流**/
    RC202("202","热点参数限流,请稍后再试!"),
    /**系统规则不满足**/
    RC203("203","系统规则不满足要求,请稍后再试!"),
    /**授权规则不通过**/
    RC204("204","授权规则不通过,请稍后再试!"),
    /**access_denied**/
    RC403("403","无访问权限,请联系管理员授予权限"),
    /**access_denied**/
    RC401("401","匿名用户访问无权限资源时的异常"),
    RC404("404","404页面找不到的异常"),
    /**服务异常**/
    RC500("500","系统异常，请稍后重试"),
    RC375("375","数学运算异常，请稍后重试"),

    INVALID_TOKEN("2001","访问令牌不合法"),
    ACCESS_DENIED("2003","没有权限访问该资源"),
    CLIENT_AUTHENTICATION_FAILED("1001","客户端认证失败"),
    USERNAME_OR_PASSWORD_ERROR("1002","用户名或密码错误"),
    BUSINESS_ERROR("1004","业务逻辑异常"),
    UNSUPPORTED_GRANT_TYPE("1003", "不支持的认证模式");

    /**自定义状态码**/
    private final String code;
    /**自定义描述**/
    private final String message;

    ReturnCodeEnum(String code, String message){
        this.code = code;
        this.message = message;
    }

    //遍历枚举V1
    public static ReturnCodeEnum getReturnCodeEnum(String code)
    {
        for (ReturnCodeEnum element : ReturnCodeEnum.values()) {
            if(element.getCode().equalsIgnoreCase(code))
            {
                return element;
            }
        }
        return null;
    }
    //遍历枚举V2
    public static ReturnCodeEnum getReturnCodeEnumV2(String code)
    {
        return Arrays.stream(ReturnCodeEnum.values()).filter(x -> x.getCode().equalsIgnoreCase(code)).findFirst().orElse(null);
    }
}
```

**新建统一定义返回对象ResultData**

```plain
package com.atguigu.cloud.resp;

import lombok.Data;
import lombok.experimental.Accessors;

/**
 * @auther zzyy
 * @create 2023-11-04 11:59
 */
@Data
@Accessors(chain = true)
public class ResultData<T> {

    private String code;/** 结果状态 ,具体状态码参见枚举类ReturnCodeEnum.java*/
    private String message;
    private T data;
    private long timestamp ;


    public ResultData (){
        this.timestamp = System.currentTimeMillis();
    }

    public static <T> ResultData<T> success(T data) {
        ResultData<T> resultData = new ResultData<>();
        resultData.setCode(ReturnCodeEnum.RC200.getCode());
        resultData.setMessage(ReturnCodeEnum.RC200.getMessage());
        resultData.setData(data);
        return resultData;
    }

    public static <T> ResultData<T> fail(String code, String message) {
        ResultData<T> resultData = new ResultData<>();
        resultData.setCode(code);
        resultData.setMessage(message);

        return resultData;
    }

}
```

**修改PayController**

```plain
package com.atguigu.cloud.controller;

import com.atguigu.cloud.entities.Pay;
import com.atguigu.cloud.entities.PayDTO;
import com.atguigu.cloud.resp.ResultData;
import com.atguigu.cloud.service.PayService;
import io.swagger.v3.oas.annotations.Operation;
import io.swagger.v3.oas.annotations.tags.Tag;
import jakarta.annotation.Resource;
import org.springframework.beans.BeanUtils;
import org.springframework.web.bind.annotation.*;

/**
 * @auther zzyy
 * @create 2023-11-12 22:34
 */
@RestController
@Tag(name = "支付微服务模块",description = "支付CRUD")
public class PayController
{
    @Resource
    PayService payService;

    @PostMapping(value = "/pay/add")
    @Operation(summary = "新增",description = "新增支付流水方法,json串做参数")
    public ResultData<String> addPay(@RequestBody Pay pay)
    {
        System.out.println(pay.toString());
        int i = payService.add(pay);
        return ResultData.success("成功插入记录，返回值："+i);
    }

    @DeleteMapping(value = "/pay/del/{id}")
    @Operation(summary = "删除",description = "删除支付流水方法")
    public ResultData<Integer> deletePay(@PathVariable("id") Integer id) {
        int i = payService.delete(id);
        return ResultData.success(i);
    }

    @PutMapping(value = "/pay/update")
    @Operation(summary = "修改",description = "修改支付流水方法")
    public ResultData<String> updatePay(@RequestBody PayDTO payDTO)
    {
        Pay pay = new Pay();
        BeanUtils.copyProperties(payDTO, pay);

        int i = payService.update(pay);
        return ResultData.success("成功修改记录，返回值："+i);
    }

    @GetMapping(value = "/pay/get/{id}")
    @Operation(summary = "按照ID查流水",description = "查询支付流水方法")
    public ResultData<Pay> getById(@PathVariable("id") Integer id)
    {
        Pay pay = payService.getById(id);
        return ResultData.success(pay);
    }

    //全部查询getall作为家庭作业
}
```

**测试**

[http://localhost:8001/pay/get/1](http://localhost:8001/pay/get/1)

**结论**

通过ResultData.success()对返回结果进行包装后返回给前端

优化驱动力 ????

**查询方法写个bug**

```plain
if(id == -4) throw new RuntimeException("id不能为负数");
```

##### <!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887030521-0c7d685c-11f0-4e2a-871d-cfe797761f64.png)
##### 3.5.2.3 解决：全局异常接入返回的标准格式
**为什么需要全局异常处理器**

不用再手写try。。。catch

当然，如果非要trycf也是可以的。

**新建全局异常类GlobalExceptionHandler**

```plain
package com.atguigu.cloud.exp;

import com.atguigu.cloud.resp.ResultData;
import com.atguigu.cloud.resp.ReturnCodeEnum;
import lombok.extern.slf4j.Slf4j;
import org.springframework.http.HttpStatus;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.ResponseStatus;
import org.springframework.web.bind.annotation.RestControllerAdvice;

/**
 * @auther zzyy
 * @create 2023-11-04 12:20
 */
@Slf4j
@RestControllerAdvice
public class GlobalExceptionHandler
{
    /**
     * 默认全局异常处理。
     * @param e the e
     * @return ResultData
     */
    @ExceptionHandler(RuntimeException.class)
    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    public ResultData<String> exception(Exception e) {
        System.out.println("----come in GlobalExceptionHandler");
        log.error("全局异常信息exception:{}", e.getMessage(), e);
        return ResultData.fail(ReturnCodeEnum.RC500.getCode(),e.getMessage());
    }
}
```

**修改Controller**

```plain
package com.atguigu.cloud.controller;

import com.atguigu.cloud.entities.Pay;
import com.atguigu.cloud.entities.PayDTO;
import com.atguigu.cloud.resp.ResultData;
import com.atguigu.cloud.resp.ReturnCodeEnum;
import com.atguigu.cloud.service.PayService;
import io.swagger.v3.oas.annotations.Operation;
import io.swagger.v3.oas.annotations.tags.Tag;
import jakarta.annotation.Resource;
import org.springframework.beans.BeanUtils;
import org.springframework.web.bind.annotation.*;

/**
 * @auther zzyy
 * @create 2023-12-12 22:34
 */
@RestController
@Tag(name = "支付微服务模块",description = "支付CRUD")
public class PayController
{
    @Resource
    PayService payService;

    @PostMapping(value = "/pay/add")
    @Operation(summary = "新增",description = "新增支付流水方法,json串做参数")
    public ResultData<String> addPay(@RequestBody Pay pay)
    {
        System.out.println(pay.toString());
        int i = payService.add(pay);
        return ResultData.success("成功插入记录，返回值："+i);
    }

    @DeleteMapping(value = "/pay/del/{id}")
    @Operation(summary = "删除",description = "删除支付流水方法")
    public ResultData<Integer> deletePay(@PathVariable("id") Integer id) {
        int i = payService.delete(id);
        return ResultData.success(i);
    }

    @PutMapping(value = "/pay/update")
    @Operation(summary = "修改",description = "修改支付流水方法")
    public ResultData<String> updatePay(@RequestBody PayDTO payDTO)
    {
        Pay pay = new Pay();
        BeanUtils.copyProperties(payDTO, pay);

        int i = payService.update(pay);
        return ResultData.success("成功修改记录，返回值："+i);
    }

    @GetMapping(value = "/pay/get/{id}")
    @Operation(summary = "按照ID查流水",description = "查询支付流水方法")
    public ResultData<Pay> getById(@PathVariable("id") Integer id)
    {
        if(id == -4) throw new RuntimeException("id不能为负数");

        Pay pay = payService.getById(id);
        return ResultData.success(pay);
    }

    //全部查询getall作为家庭作业

    @RequestMapping(value = "/pay/error",method = RequestMethod.GET)
    public ResultData<Integer> getPayError()
    {
        Integer i = Integer.valueOf(200);
        try
        {
            System.out.println("--------come here");
            int data = 10/0;
        }catch (Exception e){
            e.printStackTrace();
            return ResultData.fail(ReturnCodeEnum.RC500.getCode(),e.getMessage());
        }
        return ResultData.success(i);
    }
}
```

#### 3.5.3 目前工程目录结构
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887030631-96512719-f78a-4251-9a91-679b71ee3ed7.png)

### 3.6 引入微服务理念，从这里开始
**订单微服务80如何才能调用到支付微服务8001？**

#### 3.6.1 cloud-consumer-order80微服务调用者订单Module模块
##### 3.6.1.1 建cloud-consumer-order80
##### 3.6.1.2 改POM
```plain
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>com.atguigu.cloud</groupId>
        <artifactId>mscloudV5</artifactId>
        <version>1.0-SNAPSHOT</version>
    </parent>

    <artifactId>cloud-consumer-order80</artifactId>

    <properties>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>


    <dependencies>
        <!--web + actuator-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <!--lombok-->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <!--hutool-all-->
        <dependency>
            <groupId>cn.hutool</groupId>
            <artifactId>hutool-all</artifactId>
        </dependency>
        <!--fastjson2-->
        <dependency>
            <groupId>com.alibaba.fastjson2</groupId>
            <artifactId>fastjson2</artifactId>
        </dependency>
        <!-- swagger3 调用方式 http://你的主机IP地址:5555/swagger-ui/index.html -->
        <dependency>
            <groupId>org.springdoc</groupId>
            <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```

##### 3.6.1.3 写YML
```plain
server:
  port: 80
```

##### 3.6.1.4 主启动(修改Main类名为Main80)
```plain
package com.atguigu.cloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class Main80
{
    public static void main(String[] args)
    {
        SpringApplication.run(Main80.class,args);
    }
}
```

##### 3.6.1.5 业务类
**entities**

传递数值PayDTO

```plain
package com.atguigu.cloud.entities;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

import java.io.Serializable;
import java.math.BigDecimal;

/**
 * 一般而言，调用者不应该获悉服务提供者的entity资源并知道表结构关系，所以服务
   提供方给出的接口文档都都应成为DTO
 */
@Data
@AllArgsConstructor
@NoArgsConstructor
public class PayDTO implements Serializable
{
    private Integer id;
    //支付流水号
    private String payNo;
    //订单流水号
    private String orderNo;
    //用户账号ID
    private Integer userId;
    //交易金额
    private BigDecimal amount;
}
```

**ResultData统一返回值也从8001拷贝进来**

**RestTemplate**

**是什么**

RestTemplate提供了多种便捷访问远程Http服务的方法， 

是一种简单便捷的访问restful服务模板类，是Spring提供的用于访问Rest服务的客户端模板工具集

**官网地址**

[RestTemplate (Spring Framework 6.0.11 API)](https://docs.spring.io/spring-framework/docs/6.0.11/javadoc-api/org/springframework/web/client/RestTemplate.html)

**常用API使用说明**

**使用说明**

使用

使用restTemplate访问restful接口非常的简单粗暴无脑。

(url, requestMap, ResponseBean.class)这三个参数分别代表 

REST请求地址、请求参数、HTTP响应转换被转换成的对象类型。

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887030875-5025a5a0-403a-4467-83f0-8df397ce4491.png)

**getForObject方法/getForEntity方法**

返回对象为响应体中数据转化成的对象，基本上可以理解为Json

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887031492-62143059-918f-46c0-ac7b-54e62c985be9.png)

返回对象为ResponseEntity对象，包含了响应中的一些重要信息，比如响应头、响应状态码、响应体等

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887032280-d8bfa5d6-9970-45c7-9026-16213ca3cfee.png)

**postForObject/postForEntity**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887032100-38ad76eb-1e2a-46a6-9eea-9a884138c72f.png)

**GET请求方法**

```plain
<T> T getForObject(String url, Class<T> responseType, Object... uriVariables);


<T> T getForObject(String url, Class<T> responseType, Map<String, ?> uriVariables);


<T> T getForObject(URI url, Class<T> responseType);


<T> ResponseEntity<T> getForEntity(String url, Class<T> responseType, Object... uriVariables);


<T> ResponseEntity<T> getForEntity(String url, Class<T> responseType, Map<String, ?> uriVariables);


<T> ResponseEntity<T> getForEntity(URI var1, Class<T> responseType);
```

**POST请求方法**

```plain
<T> T postForObject(String url, @Nullable Object request, Class<T> responseType, Object... uriVariables);


<T> T postForObject(String url, @Nullable Object request, Class<T> responseType, Map<String, ?> uriVariables);


<T> T postForObject(URI url, @Nullable Object request, Class<T> responseType);


<T> ResponseEntity<T> postForEntity(String url, @Nullable Object request, Class<T> responseType, Object... uriVariables);
 

<T> ResponseEntity<T> postForEntity(String url, @Nullable Object request, Class<T> responseType, Map<String, ?> uriVariables);


<T> ResponseEntity<T> postForEntity(URI url, @Nullable Object request, Class<T> responseType);
```

**config配置类**

```plain
package com.atguigu.cloud.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.client.RestTemplate;

@Configuration
public class RestTemplateConfig
{
    @Bean
    public RestTemplate restTemplate()
    {
        return new RestTemplate();
    }
}
```

**controller**

```plain
package com.atguigu.cloud.controller;

import com.atguigu.cloud.entities.PayDTO;
import com.atguigu.cloud.resp.ResultData;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.client.RestTemplate;

@RestController
public class OrderController{
    public static final String PaymentSrv_URL = "http://localhost:8001";//先写死，硬编码
    @Autowired
    private RestTemplate restTemplate;

    /**
     * 一般情况下，通过浏览器的地址栏输入url，发送的只能是get请求
     * 我们底层调用的是post方法，模拟消费者发送get请求，客户端消费者
     * 参数可以不添加@RequestBody
     * @param payDTO
     * @return
     */
    @GetMapping("/consumer/pay/add")
    public ResultData addOrder(PayDTO payDTO){
        return restTemplate.postForObject(PaymentSrv_URL + "/pay/add",payDTO,ResultData.class);
    }
    // 删除+修改操作作为家庭作业，O(∩_∩)O。。。。。。。
    @GetMapping("/consumer/pay/get/{id}")
    public ResultData getPayInfo(@PathVariable("id") Integer id){
        return restTemplate.getForObject(PaymentSrv_URL + "/pay/get/"+id, ResultData.class, id);
    }
}
```

##### 3.6.1.6 Postman测试
[http://localhost/consumer/pay/get/1](http://localhost/consumer/pay/get/1)

[http://localhost/consumer/pay/add?payNo=1213&orderNo=1213&userId=2&amount=3.33](http://localhost/consumer/pay/add?payNo=1213&orderNo=1213&userId=2&amount=3.33)

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887033558-dac2f30a-b531-4f08-81e6-af5e0b8dda27.png)

#### 3.6.2 工程重构
##### 3.6.2.1 观察问题
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887032925-2f521111-1b8b-4eba-9fa8-c57da055cb0b.png)

系统中有重复部分，重构 

##### 3.6.2.2 新建Module
cloud-api-commons

对外暴露通用的组件/api/接口/工具类等

##### 3.6.2.3 改POM
```plain
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>com.atguigu.cloud</groupId>
        <artifactId>mscloudV6</artifactId>
        <version>1.0-SNAPSHOT</version>
    </parent>

    <artifactId>cloud-api-commons</artifactId>

    <properties>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>


    <dependencies>
        <!--SpringBoot通用依赖模块-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <!--hutool-->
        <dependency>
            <groupId>cn.hutool</groupId>
            <artifactId>hutool-all</artifactId>
        </dependency>
    </dependencies>

</project>
```

##### 3.6.2.4 entities
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887033061-bf6e0727-2b94-4de4-818a-856e64935b77.png)

PayDTO

统一返回

##### 3.6.2.5 全局异常类，可加可不加，酌情
##### 3.6.2.6 maven命令clean install
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887033486-92fa20d3-c5f5-4182-a907-a0ec5f74f25c.png)

##### 3.6.2.7 订单80和支付8001分别改造
**删除各自的原先有过的entities和统一返回体等内容**

**各自粘贴POM内容**

```plain
<!-- 引入自己定义的api通用包 -->
<dependency>
    <groupId>com.atguigu.cloud</groupId>
    <artifactId>cloud-api-commons</artifactId>
    <version>1.0-SNAPSHOT</version>
</dependency>
```

**80**

```plain
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>com.atguigu.cloud</groupId>
        <artifactId>mscloudV5</artifactId>
        <version>1.0-SNAPSHOT</version>
    </parent>

    <artifactId>cloud-consumer-order80</artifactId>

    <properties>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>


    <dependencies>
        <!-- 引入自己定义的api通用包 -->
        <dependency>
            <groupId>com.atguigu.cloud</groupId>
            <artifactId>cloud-api-commons</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
        <!--web + actuator-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <!--lombok-->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <!--hutool-all-->
        <dependency>
            <groupId>cn.hutool</groupId>
            <artifactId>hutool-all</artifactId>
        </dependency>
        <!--fastjson2-->
        <dependency>
            <groupId>com.alibaba.fastjson2</groupId>
            <artifactId>fastjson2</artifactId>
        </dependency>
        <!-- swagger3 调用方式 http://你的主机IP地址:5555/swagger-ui/index.html -->
        <dependency>
            <groupId>org.springdoc</groupId>
            <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```

**8001**

```plain
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>com.atguigu.cloud</groupId>
        <artifactId>mscloudV5</artifactId>
        <version>1.0-SNAPSHOT</version>
    </parent>

    <artifactId>cloud-provider-payment8001</artifactId>

    <properties>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>


    <dependencies>
        <!-- 引入自己定义的api通用包 -->
        <dependency>
            <groupId>com.atguigu.cloud</groupId>
            <artifactId>cloud-api-commons</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
        <!--SpringBoot通用依赖模块-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <!--SpringBoot集成druid连接池-->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid-spring-boot-starter</artifactId>
        </dependency>
        <!-- Swagger3 调用方式 http://你的主机IP地址:5555/swagger-ui/index.html -->
        <dependency>
            <groupId>org.springdoc</groupId>
            <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
        </dependency>
        <!--mybatis和springboot整合-->
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
        </dependency>
        <!--Mysql数据库驱动8 -->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
        </dependency>
        <!--persistence-->
        <dependency>
            <groupId>javax.persistence</groupId>
            <artifactId>persistence-api</artifactId>
        </dependency>
        <!--通用Mapper4-->
        <dependency>
            <groupId>tk.mybatis</groupId>
            <artifactId>mapper</artifactId>
        </dependency>
        <!--hutool-->
        <dependency>
            <groupId>cn.hutool</groupId>
            <artifactId>hutool-all</artifactId>
        </dependency>
        <!-- fastjson2 -->
        <dependency>
            <groupId>com.alibaba.fastjson2</groupId>
            <artifactId>fastjson2</artifactId>
        </dependency>
        <!--lombok-->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.18.28</version>
            <scope>provided</scope>
        </dependency>
        <!--test-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
```

##### 3.6.2.8 postman测试
[http://localhost/consumer/pay/get/1](http://localhost/consumer/pay/get/1)

[http://localhost/consumer/pay/add?payNo=1213&orderNo=1213&userId=2&amount=3.33](http://localhost/consumer/pay/add?payNo=1213&orderNo=1213&userId=2&amount=3.33)

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887033278-68525696-f25b-4a3e-847c-581cfe2859ef.png)

#### 3.6.3 目前工程样图
到这里项目必须全部做对做成功，有问题的同学给我发送邮件**zzyybs@126.com**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887034168-c52e2ebf-e611-4f72-9d47-8079619ff1c2.png)

#### 3.6.4 为什么要引入微服务
上一步controller问题？？？

硬编码写死问题

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887034369-779f0b22-b381-4d12-a788-a636ddc48d2f.png)

微服务所在的IP地址和端口号硬编码到订单微服务中，会存在非常多的问题

（1）如果订单微服务和支付微服务的IP地址或者端口号发生了变化，则支付微服务将变得不可用，需要同步修改订单微服务中调用支付微服务的IP地址和端口号。

（2）如果系统中提供了多个订单微服务和支付微服务，则无法实现微服务的负载均衡功能。

（3）如果系统需要支持更高的并发，需要部署更多的订单微服务和支付微服务，硬编码订单微服务则后续的维护会变得异常复杂。

所以，在微服务开发的过程中，需要引入服务治理功能，实现微服务之间的动态注册与发现，从此刻开始我们正式进入SpringCloud实战

## 4 Consul服务注册与发现
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887061075-4a0c93ec-c3d6-4aed-af3a-00c3b40aa2eb.png)

### 4.1 为什么要引入服务注册中心
#### 4.1.1 为什么引入
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887060675-06b068bc-ea66-49c2-9c49-b893b2dafcfb.png)

微服务所在的IP地址和端口号硬编码到订单微服务中，会存在非常多的问题

（1）如果订单微服务和支付微服务的IP地址或者端口号发生了变化，则支付微服务将变得不可用，需要同步修改订单微服务中调用支付微服务的IP地址和端口号。

（2）如果系统中提供了多个订单微服务和支付微服务，则无法实现微服务的负载均衡功能。

（3）如果系统需要支持更高的并发，需要部署更多的订单微服务和支付微服务，硬编码订单微服务则后续的维护会变得异常复杂。

所以，在微服务开发的过程中，需要引入服务治理功能，实现微服务之间的动态注册与发现，从此刻开始我们正式进入SpringCloud实战

### 4.2 为什么不再使用传统老牌的Eureka
#### 4.2.1 Eureka停更进维
[Home · Netflix/eureka Wiki · GitHub](https://github.com/Netflix/eureka/wiki)

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887060748-920a692c-e486-4b12-832f-260c1245fdda.png)

#### 4.2.2 Eureka对初学者不友好
首次看到自我保护机制

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887061023-b796012a-1a96-4dd7-aa09-640eb27709ed.png)

#### 4.2.3 注册中心独立且和微服务功能解耦
目前主流服务中心，希望单独隔离出来而不是作为一个独立微服务嵌入到系统中

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887061710-014bd9d8-2587-4453-aaef-df53a98b7ad9.png)

按照Netflix的之前的思路，注册中心Eureka也是作为一个微服务且需要程序员自己开发部署；

实际情况，

希望微服务和注册中心分离解耦，注册中心和业务无关的，不要混为一谈。

提供类似tomcat一样独立的组件，微服务注册上去使用，是个成品。

#### 4.2.4 阿里巴巴Nacos的崛起
Service discovery and configuration management

### 4.3 consul简介
#### 4.3.1 是什么
consul官网地址：[Consul by HashiCorp](https://www.consul.io/)

**What is Consul?**

Consul 是一套开源的分布式服务发现和配置管理系统，由 HashiCorp 公司用 Go 语言开发。

提供了微服务系统中的服务治理、配置中心、控制总线等功能。这些功能中的每一个都可以根据需要单独使用，也可以一起使用以构建全方位的服务网格，总之Consul提供了一种完整的服务网格解决方案。它具有很多优点。包括： 基于 raft 协议，比较简洁； 支持健康检查, 同时支持 HTTP 和 DNS 协议 支持跨数据中心的 WAN 集群 提供图形界面 跨平台，支持 Linux、Mac、Windows

[What is Consul? | Consul | HashiCorp Developer](https://developer.hashicorp.com/consul/docs/intro)

**spring consul**

[Spring Cloud Consul](https://docs.spring.io/spring-cloud-consul/docs/current/reference/html/)

#### 4.3.2 能干嘛
Consul 具有如下特性：

+ **服务发现**
    - 提供HTTP和DNS两种发现方式。
+ **健康监测**
    - 支持多种方式，HTTP、TCP、Docker、Shell脚本定制化监控
+ **KV存储**
    - Key、Value的存储方式
+ **多数据中心**
    - Consul支持多数据中心

**可视化Web界面 **

#### 4.3.3 去哪下
[Install | Consul | HashiCorp Developer](https://developer.hashicorp.com/consul/downloads)

#### 4.3.4 怎么玩
[Spring Cloud Consul](https://docs.spring.io/spring-cloud-consul/docs/current/reference/html/)

两大作用

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887064230-8f21d599-a511-46e7-b5af-e3a0bd5f38fc.png)

### 4.4 安装并运行consul
**官网下载**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887064764-3f0b788b-319b-4b1c-a2a1-320faf55a4ad.png)

[Install | Consul | HashiCorp Developer](https://developer.hashicorp.com/consul/downloads)

**下载完成后只有一个consul.exe文件，对应全路径下查看版本号信息**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887065206-4350f4da-c43a-4e4b-a8bd-3225ec057da0.png)

**使用开发模式启动**

**consul agent -dev**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887065712-4718dc6e-ea5a-4dc0-82e1-80179868e25c.png)

**通过以下地址可以访问Consul的首页：http://localhost:8500**

**结果页面**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887066085-d81506d5-b0f6-4efd-a004-a5dac5a9a6eb.png)

### 4.5 服务注册与发现
#### 4.5.1 服务提供者8001
**支付服务provider8001注册进consul**

**POM**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <parent>
    <groupId>com.atguigu.cloud</groupId>
    <artifactId>mscloudV5</artifactId>
    <version>1.0-SNAPSHOT</version>
  </parent>

  <artifactId>cloud-provider-payment8001</artifactId>

  <properties>
    <maven.compiler.source>17</maven.compiler.source>
    <maven.compiler.target>17</maven.compiler.target>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
  </properties>


  <dependencies>
    <!--SpringCloud consul discovery -->
    <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-consul-discovery</artifactId>
    </dependency>
    <!-- 引入自己定义的api通用包 -->
    <dependency>
      <groupId>com.atguigu.cloud</groupId>
      <artifactId>cloud-api-commons</artifactId>
      <version>1.0-SNAPSHOT</version>
    </dependency>
    <!--SpringBoot通用依赖模块-->
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
    <!--SpringBoot集成druid连接池-->
    <dependency>
      <groupId>com.alibaba</groupId>
      <artifactId>druid-spring-boot-starter</artifactId>
    </dependency>
    <!-- Swagger3 调用方式 http://你的主机IP地址:5555/swagger-ui/index.html -->
    <dependency>
      <groupId>org.springdoc</groupId>
      <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
    </dependency>
    <!--mybatis和springboot整合-->
    <dependency>
      <groupId>org.mybatis.spring.boot</groupId>
      <artifactId>mybatis-spring-boot-starter</artifactId>
    </dependency>
    <!--Mysql数据库驱动8 -->
    <dependency>
      <groupId>mysql</groupId>
      <artifactId>mysql-connector-java</artifactId>
    </dependency>
    <!--persistence-->
    <dependency>
      <groupId>javax.persistence</groupId>
      <artifactId>persistence-api</artifactId>
    </dependency>
    <!--通用Mapper4-->
    <dependency>
      <groupId>tk.mybatis</groupId>
      <artifactId>mapper</artifactId>
    </dependency>
    <!--hutool-->
    <dependency>
      <groupId>cn.hutool</groupId>
      <artifactId>hutool-all</artifactId>
    </dependency>
    <!-- fastjson2 -->
    <dependency>
      <groupId>com.alibaba.fastjson2</groupId>
      <artifactId>fastjson2</artifactId>
    </dependency>
    <!--lombok-->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.18.28</version>
            <scope>provided</scope>
        </dependency>
        <!--test-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
```

配置来源

[Quick Start :: Spring Cloud Consul](https://docs.spring.io/spring-cloud-consul/reference/quickstart.html)

**YML**

```yaml
server:
  port: 8001

# ==========applicationName + druid-mysql8 driver===================
spring:
  application:
    name: cloud-payment-service

  datasource:
    type: com.alibaba.druid.pool.DruidDataSource
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/db2024?characterEncoding=utf8&useSSL=false&serverTimezone=GMT%2B8&rewriteBatchedStatements=true&allowPublicKeyRetrieval=true
    username: root
    password: 123456
  ####Spring Cloud Consul for Service Discovery
  cloud:
    consul:
      host: localhost
      port: 8500
      discovery:
        service-name: ${spring.application.name}

# ========================mybatis===================
mybatis:
  mapper-locations: classpath:mapper/*.xml
  type-aliases-package: com.atguigu.cloud.entities
  configuration:
    map-underscore-to-camel-case: true
```

**主启动**

```java
package com.atguigu.cloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import tk.mybatis.spring.annotation.MapperScan;

/**
 * @auther zzyy
 * @create 2023-11-03 17:54
 */
@SpringBootApplication
@MapperScan("com.atguigu.cloud.mapper") //import tk.mybatis.spring.annotation.MapperScan;
@EnableDiscoveryClient
public class Main8001{
    public static void main(String[] args){
        SpringApplication.run(Main8001.class,args);
    }
}
```

`@EnableDiscoveryClient `开启服务发现

**启动8001并查看consul控制台**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887066029-866889bf-8df2-4f1d-95da-74e1e1ca5313.png)

#### 4.5.2 服务消费者80
**修改微服务cloud-consumer-order80**

**POM**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <parent>
    <groupId>com.atguigu.cloud</groupId>
    <artifactId>mscloudV5</artifactId>
    <version>1.0-SNAPSHOT</version>
  </parent>

  <artifactId>cloud-consumer-order80</artifactId>

  <properties>
    <maven.compiler.source>17</maven.compiler.source>
    <maven.compiler.target>17</maven.compiler.target>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
  </properties>


  <dependencies>
    <!--SpringCloud consul discovery -->
    <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-consul-discovery</artifactId>
    </dependency>
    <!-- 引入自己定义的api通用包 -->
    <dependency>
      <groupId>com.atguigu.cloud</groupId>
      <artifactId>cloud-api-commons</artifactId>
      <version>1.0-SNAPSHOT</version>
    </dependency>
    <!--web + actuator-->
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
    <!--lombok-->
    <dependency>
      <groupId>org.projectlombok</groupId>
      <artifactId>lombok</artifactId>
      <optional>true</optional>
    </dependency>
    <!--hutool-all-->
    <dependency>
      <groupId>cn.hutool</groupId>
      <artifactId>hutool-all</artifactId>
    </dependency>
    <!--fastjson2-->
    <dependency>
      <groupId>com.alibaba.fastjson2</groupId>
      <artifactId>fastjson2</artifactId>
    </dependency>
    <!-- swagger3 调用方式 http://你的主机IP地址:5555/swagger-ui/index.html -->
    <dependency>
      <groupId>org.springdoc</groupId>
      <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
    </dependency>
  </dependencies>

  <build>
    <plugins>
      <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
      </plugin>
    </plugins>
  </build>
</project>
```

**YML**

```yaml
server:
  port: 80

spring:
  application:
    name: cloud-consumer-order
  ####Spring Cloud Consul for Service Discovery
  cloud:
    consul:
      host: localhost
      port: 8500
      discovery:
        prefer-ip-address: true #优先使用服务ip进行注册
        service-name: ${spring.application.name}
```

**主启动类**

```java
package com.atguigu.cloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;

/**
 * @auther zzyy
 * @create 2023-11-04 15:19
 */
@SpringBootApplication
@EnableDiscoveryClient //该注解用于向使用consul为注册中心时注册服务
public class Main80{
    public static void main(String[] args){
        SpringApplication.run(Main80.class,args);
    }
}
```

**Controller**

```java
package com.atguigu.cloud.controller;

import com.atguigu.cloud.entities.PayDTO;
import com.atguigu.cloud.resp.ResultData;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.client.RestTemplate;

/**
 * @auther zzyy
 * @create 2023-11-04 16:00
 */
@RestController
public class OrderController{
    //public static final String PaymentSrv_URL = "http://localhost:8001";//先写死，硬编码

    public static final String PaymentSrv_URL = "http://cloud-payment-service";//服务注册中心上的微服务名称

    @Resource
    private RestTemplate restTemplate;

    /**
     * 一般情况下，通过浏览器的地址栏输入url，发送的只能是get请求
     * 我们模拟消费者发送get请求，but底层调用post方法，客户端消费者参数PayDTO可以不添加@RequestBody
     * @param payDTO
     * @return
     */
    @GetMapping("/consumer/pay/add")
    public ResultData addOrder(PayDTO payDTO){
        return restTemplate.postForObject(PaymentSrv_URL + "/pay/add",payDTO,ResultData.class);
    }

    // 删除+修改操作作为家庭作业，O(∩_∩)O。。。。。。。

    @GetMapping("/consumer/pay/get/{id}")
    public ResultData getPayInfo(@PathVariable Integer id){
        return restTemplate.getForObject(PaymentSrv_URL + "/pay/get/"+id, ResultData.class, id);
    }
}
```

**启动80并查看consul控制台**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887066489-6ec0a14d-ec2d-4532-a4e0-7c21c9f2e071.png)

**结果如何**

一个bug

java.net.UnknownHostException: cloud-payment-service

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887067992-e2692209-c3e4-46e5-b07b-a28de12be9e8.png)

**配置修改RestTemplateConfig**

```java
package com.atguigu.cloud.config;

import org.springframework.cloud.client.loadbalancer.LoadBalanced;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.client.RestTemplate;

/**
 * @auther zzyy
 * @create 2023-11-04 15:57
 */
@Configuration
public class RestTemplateConfig{
    
    @Bean
    @LoadBalanced
    public RestTemplate restTemplate(){
        return new RestTemplate();
    }
}
```

#### 4.5.3 三个注册中心异同点
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887068458-2e23c7df-4677-47c8-9dde-752883ef44f1.png)

**CAP**

C:Consistency（强一致性）

A:Availability（可用性）

P:Partition tolerance（分区容错性）

**经典CAP图**

最多只能同时较好的满足两个。

CAP理论的核心是：一个分布式系统不可能同时很好的满足一致性，可用性和分区容错性这三个需求，

因此，根据 CAP 原理将 NoSQL 数据库分成了满足 CA 原则、满足 CP 原则和满足 AP 原则三 大类：

CA - 单点集群，满足一致性，可用性的系统，通常在可扩展性上不太强大。

CP - 满足一致性，分区容忍必的系统，通常性能不是特别高。

AP - 满足可用性，分区容忍性的系统，通常可能对一致性要求低一些。

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887068315-ca9a9b0f-0db9-4a72-8cd2-cac04234211f.png)

**AP(Eureka)**

AP架构

当网络分区出现后，为了保证可用性，系统B可以返回旧值，保证系统的可用性。

当数据出现不一致时，虽然A, B上的注册信息不完全相同，但每个Eureka节点依然能够正常对外提供服务，这会出现查询服务信息时如果请求A查不到，但请求B就能查到。如此保证了可用性但牺牲了一致性结论：违背了一致性C的要求，只满足可用性和分区容错，即AP

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887068826-ea635714-3411-4878-bff9-a04633360495.png)

**CP(Zookeeper/Consul)**

CP架构

当网络分区出现后，为了保证一致性，**就必须拒接请求**，否则无法保证一致性，Consul 遵循CAP原理中的CP原则，保证了强一致性和分区容错性，且使用的是Raft算法，比zookeeper使用的Paxos算法更加简单。虽然保证了强一致性，但是可用性就相应下降了，例如服务注册的时间会稍长一些，因为 Consul 的 raft 协议要求必须过半数的节点都写入成功才认为注册成功 ；在leader挂掉了之后，重新选举出leader之前会导致Consul 服务不可用。结论：违背了可用性A的要求，只满足一致性和分区容错，即CP

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887068346-a78cba01-99f9-4197-a912-b5a6e043734e.png)

### 4.6 服务配置与刷新
#### 4.6.1 分布式系统面临的 → 配置问题
微服务意味着要将单体应用中的业务拆分成一个个子服务，每个服务的粒度相对较小，因此系统中会出现大量的服务。由于每个服务都需要必要的配置信息才能运行，所以一套集中式的、动态的配置管理设施是必不可少的。比如某些配置文件中的内容大部分都是相同的，只有个别的配置项不同。就拿数据库配置来说吧，如果每个微服务使用的技术栈都是相同的，则每个微服务中关于数据库的配置几乎都是相同的，有时候主机迁移了，我希望一次修改，处处生效。

当下我们每一个微服务自己带着一个application.yml，上百个配置文件的管理....../(ㄒoㄒ)/~~

#### 4.6.2 官网说明
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887069834-6af56218-35dc-4aff-a670-67d1e7b42997.png)

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887070835-2b43811b-7de9-434a-90d5-9a6c0db4d04f.png)

#### 4.6.3 服务配置案例步骤
##### **4.6.3.1 需求**
通用全局配置信息，直接注册进Consul服务器，从Consul获取

既然从Consul获取自然要遵守Consul的配置规则要求

##### **4.6.3.2 修改****cloud-provider-payment8001**
##### **4.6.3.3 POM**
```xml
<!--SpringCloud consul config-->
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-consul-config</artifactId>
</dependency>
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-bootstrap</artifactId>
</dependency>
```

##### **4.6.3.4 YML**
**配置规则说明**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887070535-a4e8d32a-e55c-494d-847f-1d6f970cfe38.png)

**新增配置文件bootstrap.yml**

**是什么**

applicaiton.yml是用户级的资源配置项

bootstrap.yml是系统级的，优先级更加高

Spring Cloud会创建一个“Bootstrap Context”，作为Spring应用的`Application Context`的父上下文。初始化的时候，Bootstrap Context负责从外部源加载配置属性并解析配置。这两个上下文共享一个从外部获取的`Environment`。

`Bootstrap`属性有高优先级，默认情况下，它们不会被本地配置覆盖。 `Bootstrap context`和`Application Context`有着不同的约定，所以新增了一个`bootstrap.yml`文件，保证`Bootstrap Context`和`Application Context`配置的分离。

application.yml文件改为bootstrap.yml,这是很关键的或者两者共存

因为bootstrap.yml是比application.yml先加载的。bootstrap.yml优先级高于application.yml

**bootstrap.yml**

```plain
spring:
  application:
    name: cloud-payment-service
    ####Spring Cloud Consul for Service Discovery
  cloud:
    consul:
      host: localhost
      port: 8500
      discovery:
        service-name: ${spring.application.name}
      config:
        profile-separator: '-' # default value is ","，we update '-'
        format: YAML

# config/cloud-payment-service/data
#       /cloud-payment-service-dev/data
#       /cloud-payment-service-prod/data
```

**application.yml**

```plain
server:
  port: 8001

# ==========applicationName + druid-mysql8 driver===================
spring:
  datasource:
    type: com.alibaba.druid.pool.DruidDataSource
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/db2024?characterEncoding=utf8&useSSL=false&serverTimezone=GMT%2B8&rewriteBatchedStatements=true&allowPublicKeyRetrieval=true
    username: root
    password: 123456
  profiles:
    active: dev # 多环境配置加载内容dev/prod,不写就是默认default配置

# ========================mybatis===================
mybatis:
  mapper-locations: classpath:mapper/*.xml
  type-aliases-package: com.atguigu.cloud.entities
  configuration:
    map-underscore-to-camel-case: true
```

##### **4.6.3.5 consul****服务器****key/value****配置填写**
**1 参考规则**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887070430-cf1c3347-2210-4c42-b02e-a9e5585d3bfe.png)

**2 创建config文件夹，以/结尾**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887070727-e15a5ffd-cc7b-4f91-b1be-47d6db87448a.png)

**3 config文件夹下分别创建其它3个文件夹，以/结尾**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887071387-5aec4a03-efdd-4421-bfbf-8d78b5144ea3.png)

**4 上述3个文件夹下分别创建data内容，data不再是文件夹**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887072903-093904c3-285c-4a40-82c0-10196896f808.png)

##### **4.6.3.6 controller**
```java
@Value("${server.port}")
private String port;

@GetMapping(value = "/pay/get/info")
private String getInfoByConsul(@Value("${atguigu.info}") String atguiguInfo) {
    return "atguiguInfo: "+atguiguInfo+"\t"+"port: "+port;
}
```

##### **4.6.3.7 测试**
| **spring**:    **profiles**:    **active**: dev  _# _ _多环境配置加载内容dev_ |
| :--- |
| **spring**:    **profiles**:    **active**: prod  _# _ _多环境配置加载内容prod_ |
| **spring**:    **profiles**:    **active**:   _# _ _多环境配置加载内容默认_ |


通过修改application.yml里面的激活配置部分，进行内容的验证

#### 4.6.4 动态刷新案例步骤
##### 4.6.4.1 问题
接上一步，我们在consul的dev配置分支修改了内容

马上访问，结果无效

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887072683-e5265b03-9a20-490d-a714-43e173ccf8ec.png)

| http://localhost:8001/pay/get/info |
| :--- |


会发现还是原来的内容，/(ㄒoㄒ)/~~ ，没有做到及时响应和动态刷新

##### 4.6.4.2 步骤
**@RefreshScope主启动类添加**

```plain
package com.atguigu.cloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.cloud.context.config.annotation.RefreshScope;
import tk.mybatis.spring.annotation.MapperScan;

/**
 * @auther zzyy
 * @create 2023-11-03 17:54
 */
@SpringBootApplication
@MapperScan("com.atguigu.cloud.mapper") //import tk.mybatis.spring.annotation.MapperScan;
@EnableDiscoveryClient //服务注册和发现
@RefreshScope // 动态刷新
public class Main8001
{
    public static void main(String[] args)
    {
        SpringApplication.run(Main8001.class,args);
    }
}
```

**bootstrap.yml修改下(只为教学实际别改) spring.cloud.consul.config.watch.wait-time**

**官网说明**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887073378-e011c062-8765-4713-8a1f-c65f0fe55ff5.png)

**修改步骤**

```plain
spring:
  application:
    name: cloud-payment-service
    ####Spring Cloud Consul for Service Discovery
  cloud:
    consul:
      host: localhost
      port: 8500
      discovery:
        service-name: ${spring.application.name}
      config:
        profile-separator: '-' # default value is ","，we update '-'
        format: YAML
        watch:
          wait-time: 1

# config/cloud-payment-service/data
#       /cloud-payment-service-dev/data
#       /cloud-payment-service-prod/data
```

**controller**

```plain
@Value("${server.port}")
private String port;

@GetMapping(value = "/pay/get/info")
private String getInfoByConsul(@Value("${atguigu.info}") String atguiguInfo)
{
    return "atguiguInfo: "+atguiguInfo+"\t"+"port: "+port;
}
```

#### 4.6.5 思考
**截止到这，服务配置和动态刷新全部通过，假设我重启Consul，之前的配置还在吗？**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887073872-3f0eed6f-4475-4a24-977f-dc0b0b3a8f03.png)

引出问题——Consul配置持久化......

## 5 LoadBalancer负载均衡服务调用
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887116928-bfb67e88-13fe-4836-911b-4cb4a24a16c4.png)

### 5.2 spring-cloud-loadbalancer概述

#### 5.2.2 LB负载均衡(Load Balance)是什么

负载均衡（Load Balance）就是将海量的网络请求或计算任务，通过某种算法合理地分发到多台服务器上共同处理，以防止单台服务器过载宕机，从而保证系统的高可用性、稳定性和极速响应。

**spring-cloud-starter-loadbalancer组件是什么**

**`spring-cloud-starter-loadbalancer`** 是 Spring Cloud 官方提供的一个客户端负载均衡器（Client-Side Load Balancer）组件。

简单来说，如果把 Nginx 等传统负载均衡器比作“站在门口统一分配任务的独立大堂经理”（服务端负载均衡），那么 Spring Cloud LoadBalancer 就是“直接内置在每个微服务内部的智能导航仪”。

#### 5.2.3 客户端负载 VS 服务器端负载区别

这两者的核心区别在于：**“决定把请求发给谁”的大脑长在谁身上**，以及**请求是否需要经过中间人**。

我们可以通过一个通俗的比喻和具体的对比来彻底理清它们的区别：

##### 1. 服务器端负载均衡 (Server-Side Load Balancing)

- **工作原理：** 客户端（调用方）不知道后端到底有多少台服务器。它只管把请求统一发给一个**集中的、独立的负载均衡器**（如 Nginx、F5、HAProxy）。由这个负载均衡器作为“中间人”，根据算法挑选一台后端服务器，并把请求转发过去。
    
- **生活比喻（去银行办事）：** 你（客户端）来到银行，不知道哪个柜台空闲。你只能先到大堂经理 / 取号机（服务端负载均衡器）那里，由大堂经理统筹安排，把你指派到具体的 1 号窗口或 2 号窗口（后端服务）。
    
- **特点：**
    
    - 集中式管理，对客户端完全透明（客户端觉得它只是在跟一个唯一的服务器交互）。
        
    - 多了一次网络转发（客户端 -> 负载均衡器 -> 后端服务器）。

##### 2. 客户端负载均衡 (Client-Side Load Balancing)

- **工作原理：** 客户端（调用方）自己内置了负载均衡的功能（如 Spring Cloud LoadBalancer）。它会先从**服务注册中心**（如 Nacos、Eureka）拉取所有可用目标服务器的清单，然后**自己根据算法**（如轮询、随机）挑出一台，**直接**发起网络调用。
    
- **生活比喻（用地图找餐厅）：** 你（客户端）想吃海底捞，你先打开**高德地图（服务注册中心）**搜了一下，发现附近有 3 家分店。你**自己（客户端负载均衡器）**在脑子里盘算了一下，决定去距离最近的那家，然后**直接**走过去（发起请求）。中间不需要任何人帮你转交点餐单。
    
- **特点：**
    
    - 去中心化，没有独立的中间代理服务器，避免了单点故障和性能瓶颈。
        
    - 网络直连，少了一次转发（客户端 -> 后端服务器），延迟更低。
        
    - 强依赖于服务注册中心。

### 5.3 spring-cloud-loadbalancer负载均衡解析
#### 5.3.1 负载均衡演示案例-理论
**架构说明:80通过轮询负载访问8001/8002/8003**

![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887117999-2c27ca3a-9662-4998-ad68-a80a44023b5b.png)

LoadBalancer 在工作时分成两步：

**第一步**，先选择ConsulServer从服务端查询并拉取服务列表，知道了它有多个服务(上图3个服务)，这3个实现是完全一样的，

默认轮询调用谁都可以正常执行。类似生活中求医挂号，某个科室今日出诊的全部医生，客户端你自己选一个。

**第二步**，按照指定的负载均衡策略从server取到的服务注册列表中由客户端自己选择一个地址，所以LoadBalancer是一个**客户端的**负载均衡器。

#### 5.3.2 负载均衡演示案例-实操
##### 1. 核心概念
+ **客户端负载均衡**：调用方（客户端）本地维护服务实例列表，选择合适的实例发起调用。
+ 默认策略：**Round Robin（轮询）**。
+ 支持与 **Eureka / Nacos / Consul** 等服务发现组件无缝集成。
+ 支持 RestTemplate、WebClient、Feign Client、WebFlux 等。

##### 2. 环境准备（推荐 Spring Boot 3.x + Spring Cloud 2023.x）
**依赖（pom.xml）**：

```  xml
<dependencies>
  <!-- Spring Cloud LoadBalancer -->
  <dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-loadbalancer</artifactId>
  </dependency>

  <!-- 服务发现（以 Eureka 为例，可换 Nacos） -->
  <dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
  </dependency>

  <!-- Web 调用 -->
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
  </dependency>
</dependencies>

<dependencyManagement>
  <dependencies>
    <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-dependencies</artifactId>
      <version>2023.0.3</version> <!-- 或最新版本 -->
      <type>pom</type>
      <scope>import</scope>
    </dependency>
  </dependencies>
</dependencyManagement>
```

##### 3. 启动两个后端服务实例（模拟多实例）
创建 **provider-service** 项目，端口分别设为 **8081** 和 **8082**。

**application.yml**（两个实例只改端口）：

```yaml
server:
  port: 8081   # 另一个实例改为 8082

spring:
  application:
    name: provider-service

eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
```

**Controller**：

```java
@RestController
public class HelloController {

    @GetMapping("/hello")
    public String hello() {
        return "Hello from " + System.getenv("HOSTNAME") + " - Port: " + System.getProperty("server.port") + " - Time: " + new Date();
    }
    
}
```

启动两个实例（IDEA 直接改端口启动，或打包后用不同端口运行）。

##### 4. 客户端（consumer-service）配置
**application.yml**：

```yaml
spring:
  application:
    name: consumer-service
  cloud:
    loadbalancer:
      enabled: true

eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
```

**配置 LoadBalanced RestTemplate**：

```java
@Configuration
public class RestTemplateConfig {

    @Bean
    @LoadBalanced      // 关键注解
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
    
}
```

**Service 调用**：

```java
@Service
public class ConsumerService {

    @Autowired
    private RestTemplate restTemplate;

    public String callProvider() {
        // 使用服务名称调用，LoadBalancer 自动负载均衡
        return restTemplate.getForObject("http://provider-service/hello", String.class);
    }
    
}
```

**Controller**：

```java
@RestController
@RequestMapping("/consumer")
public class ConsumerController {

    @Autowired
    private ConsumerService consumerService;

    @GetMapping("/hello")
    public String hello() {
        return consumerService.callProvider();
    }
}
```

##### 5. 测试负载均衡效果
1. 启动 Eureka Server（8761）。
2. 启动两个 provider-service 实例（8081、8082）。
3. 启动 consumer-service。
4. 浏览器或 Postman 反复访问：http://localhost:8080/consumer/hello

你会看到请求在两个后端实例之间**轮询**返回。

### 5.4 负载均衡算法原理
#### 5.4.1 默认算法是什么？有几种？

##### 一、 核心基础算法（2种）

1. **轮询 (RoundRobinLoadBalancer) —— 【默认算法】**
    
    - **原理：** 绝对的公平主义。按照目标服务器列表的顺序，挨个分配请求。第一大发给 A，第二大发给 B，第三大发给 C，第四大再发给 A，以此类推。
        
    - **适用场景：** 后端集群中所有服务器的硬件配置和处理能力基本一致的情况。
        
2. **随机 (RandomLoadBalancer)**
    
    - **原理：** 闭着眼睛瞎抓。每次请求过来，完全依靠随机数生成器从可用实例列表中抽取一个。当请求量足够大时，其实际效果会无限趋近于轮询。
        
    - **适用场景：** 比较通用的场景，或者当实例列表频繁发生变动，且不想维护轮询状态时。

##### 二、 高级路由与过滤策略

除了上面两种绝对的基础算法，Spring Cloud LoadBalancer 还通过 `ServiceInstanceListSupplier`（服务实例列表供应商）机制，提供了几种贴近实战的“筛选策略”。这些策略通常会先筛选出一批实例，然后再在这批实例里进行轮询或随机：

- **区域优先 (Zone Preference)：** 优先将请求发给与客户端处于同一个物理区域（机房/可用区）的服务器实例。这能极大地降低网络延迟并节约跨区域带宽。如果同区域没有可用实例，再跨区域调用。
    
- **同实例优先/会话保持 (Same Instance Preference)：** 如果之前某个请求成功调用了实例 A，那么后续类似的请求会尽量继续发给实例 A。这在需要维持一定本地缓存命中率的场景下很有用。
    
- **健康检查优先 (Health Check)：** 定期主动对服务实例进行健康检查（比如 ping 一下或者检查某个 actuator 接口），在分配流量前，自动把那些响应太慢或者已经挂掉的实例从列表中剔除。
    
- **基于提示的路由 (Hint Based)：** 允许在发起请求的 Header 中带上一些特定的“提示信息”（Hint），负载均衡器会根据这些提示将请求路由到特定的实例（常用于灰度发布或 A/B 测试）。

#### 5.4.2 算法切换

在 Spring Cloud LoadBalancer 中，默认是轮询算法（Round Robin）。如果我们要将其切换为随机算法（Random）或者自定义算法，可以通过代码配置来实现。

切换算法分为两种场景：**局部切换**（只针对调用某个特定的微服务）和**全局切换**（针对调用所有的微服务）。

以下是具体的代码操作步骤：

##### 第一步：编写自定义的负载均衡配置类

我们需要提供一个 `ReactorLoadBalancer` 的 Bean。这里以切换为 Spring 自带的 `RandomLoadBalancer`（随机算法）为例。
``` java
import org.springframework.cloud.client.ServiceInstance;
import org.springframework.cloud.loadbalancer.core.RandomLoadBalancer;
import org.springframework.cloud.loadbalancer.core.ReactorLoadBalancer;
import org.springframework.cloud.loadbalancer.core.ServiceInstanceListSupplier;
import org.springframework.cloud.loadbalancer.support.LoadBalancerClientFactory;
import org.springframework.context.annotation.Bean;
import org.springframework.core.env.Environment;

// 注意：这里强烈建议【不要】加 @Configuration 注解！
// 如果加了 @Configuration 且被 Spring Boot 的 @ComponentScan 扫描到，它会变成全局默认配置。
public class CustomLoadBalancerConfig {

    @Bean
    public ReactorLoadBalancer<ServiceInstance> randomLoadBalancer(
            Environment environment,
            LoadBalancerClientFactory loadBalancerClientFactory) {
        
        // 获取当前正在请求的微服务名称
        String name = environment.getProperty(LoadBalancerClientFactory.PROPERTY_NAME);
        
        // 返回随机算法的负载均衡器实例
        return new RandomLoadBalancer(
                loadBalancerClientFactory.getLazyProvider(name, ServiceInstanceListSupplier.class), 
                name
        );
    }
}
```

##### 第二步：将配置应用到微服务调用中

根据你的需求，选择局部生效还是全局生效，使用特定的注解来绑定刚才写好的配置类。通常建议配置在之前的 `RestTemplateConfig` 也就是声明 `@LoadBalanced` 的类上。

###### 场景 A：局部切换（推荐）

假设你只希望在调用 `user-service` 这个微服务时使用**随机算法**，而调用其他服务依然保持默认的**轮询算法**。

使用 `@LoadBalancerClient` 注解：

``` java
import org.springframework.cloud.client.loadbalancer.LoadBalanced;
import org.springframework.cloud.loadbalancer.annotation.LoadBalancerClient;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.client.RestTemplate;

@Configuration
// 重点在这里：name 指定目标服务名，configuration 指定你写的配置类
@LoadBalancerClient(name = "user-service", configuration = CustomLoadBalancerConfig.class) 
public class RestTemplateConfig {

    @Bean
    @LoadBalanced
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
    
}
```

###### 场景 B：全局切换

如果你希望你这个客户端项目，无论调用哪个微服务（`user-service`、`order-service` 等），统统都使用**随机算法**。

使用 `@LoadBalancerClients` 注解：

``` java
import org.springframework.cloud.client.loadbalancer.LoadBalanced;
import org.springframework.cloud.loadbalancer.annotation.LoadBalancerClients;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.client.RestTemplate;

@Configuration
// 重点在这里：defaultConfiguration 会将指定的配置应用到所有的服务调用中
@LoadBalancerClients(defaultConfiguration = CustomLoadBalancerConfig.class)
public class RestTemplateConfig {

    @Bean
    @LoadBalanced
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
}
```


## 6 OpenFeign服务接口调用

### 6.1 提问

**LoadBalancer 解决的是“怎么选机器”的问题，而 OpenFeign 解决的是“怎么让自己少写代码、让代码更优雅”的问题。**

它们绝不是替代关系，而是**上下级关系**。事实上，当你学会 OpenFeign 之后，你依然在用 LoadBalancer——因为 **OpenFeign 的底层已经默默集成了 LoadBalancer**。


### **6.3 OpenFeign能干什么**

OpenFeign 绝对不仅仅是一个“把 URL 换成接口”的换壳工具。在微服务架构中，它作为服务间通信的“全能外交官”，几乎包揽了所有与网络传输、数据解析、安全和稳定性相关的脏活累活。

具体来说，OpenFeign 在实战中核心能干以下 7 件事：

#### 1. 声明式方法调用（让远程调用回归面向对象）

这是它最直观的本领。它能把复杂的 HTTP 请求（包含 Method、URL、Header、Body）直接映射为一个普通的 Java 接口方法。你只需要像调用本地的 Service 一样调用它，完全摆脱了拼装 URL 字符串的低效工作。

#### 2. 强力的参数绑定（自动处理 JSON 与传参）

无论是简单的键值对，还是复杂的 JSON 对象，OpenFeign 都能完美支持 Spring MVC 的原生注解，并在底层自动完成 **序列化与反序列化**：

- 使用 `@RequestParam` 自动拼接 URL 参数。
    
- 使用 `@PathVariable` 自动替换路径占位符。
    
- 使用 `@RequestBody` 自动将 Java 对象转为 JSON 塞入请求体。
    
- 使用 `@RequestHeader` 动态传递自定义请求头。
    

#### 3. 无缝集成负载均衡（自动轮询机器）

OpenFeign 内部天然集成了 `Spring Cloud LoadBalancer`。 你只需要在 `@FeignClient(name = "xxx")` 中写上微服务的别名，Feign 在发起调用前，就会自动去注册中心（如 Nacos/Eureka）拉取健康的实例列表，并默认通过**轮询算法**挑出一台机器进行访问，无需任何额外配置。

#### 4. 请求拦截器（微服务全链路 Token 透传）

在微服务中，用户登录后的 JWT Token 通常保存在网关（Gateway）。当网关把请求转发给 A 服务，A 服务又需要通过 Feign 调用 B 服务时，Token 很容易在服务间传递时**丢失**。

- **OpenFeign 可以通过实现 `RequestInterceptor` 接口**，在每次发起远程调用前**统一拦截请求**，自动把当前线程上下文中的 Token、流水号（TraceId）重新塞进 Feign 的请求头里，实现全链路身份鉴权。

#### 5. 细粒度的日志审计（排查接口报错的神器）

原生的 `RestTemplate` 报错时很难看清 HTTP 请求的具体细节。OpenFeign 自带了非常强大的日志工厂，支持 4 种级别的配置：

- `NONE`：不记录任何日志（默认）。
    
- `BASIC`：仅记录请求方法、URL、响应状态码及执行时间。
    
- `HEADERS`：在 BASIC 的基础上，记录请求和响应的 Header 头信息。
    
- `FULL`：记录请求和响应的 Header、Body 以及元数据（开发调试阶段的绝佳利器）。

#### 6. 请求与响应压缩（提升传输性能）

当微服务之间需要传输大文本（如大段的业务数据或较长的 JSON 列表）时，OpenFeign 支持开启 **GZIP 压缩**。 通过在配置文件中开启 `feign.compression.request.enabled=true`，可以大幅减少网络传输的带宽占用，显著提升高并发下的接口响应速度。

#### 7. 熔断降级与容错保护（防止雪崩）

网络调用天然是不稳定的。如果被调用的 B 服务因为数据库卡死突然响应极慢，A 服务疯狂调用它就会导致 A 服务的线程堆积，最终引发整个系统的**微服务雪崩**。

- OpenFeign 提供了原生对接 `Sentinel` 或 `Resilience4j` 的接口。你可以直接在 `@FeignClient` 注解中配置 `fallback` 属性，指定一个“备胎类”。一旦对方服务超时或挂掉，Feign 会立刻秒级触发“降级方法”（比如返回一个友好的错误提示或空对象），保全大局。

### 6.4 OpenFeign通用步骤(怎么玩)
#### 6.4.1 接口+注解
**微服务Api接口+@FeignClient注解标签**

**架构说明图**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887187074-7dd8bd87-bf50-481e-8c63-554f6bf05fca.png)

| 服务消费者80 → 调用含有 @FeignClient注解的Api服务接口  → 服务提供者(8001/8002) |
| :--- |


#### 6.4.2 流程步骤
##### **6.4.2.1 建****Module**
**cloud-consumer-feign-order80**

Feign在消费端使用

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887186866-bdacbfae-a3a3-4f1e-9bc0-7e757ad133f7.png)

##### **6.4.2.2 改POM**
```plain
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>com.atguigu.cloud</groupId>
        <artifactId>mscloudV5</artifactId>
        <version>1.0-SNAPSHOT</version>
    </parent>

    <artifactId>cloud-consumer-feign-order80</artifactId>

    <properties>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>


    <dependencies>
        <!--openfeign-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>
        <!--SpringCloud consul discovery-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-consul-discovery</artifactId>
        </dependency>
        <!-- 引入自己定义的api通用包 -->
        <dependency>
            <groupId>com.atguigu.cloud</groupId>
            <artifactId>cloud-api-commons</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
        <!--web + actuator-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <!--lombok-->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <!--hutool-all-->
        <dependency>
            <groupId>cn.hutool</groupId>
            <artifactId>hutool-all</artifactId>
        </dependency>
        <!--fastjson2-->
        <dependency>
            <groupId>com.alibaba.fastjson2</groupId>
            <artifactId>fastjson2</artifactId>
        </dependency>
        <!-- swagger3 调用方式 http://你的主机IP地址:5555/swagger-ui/index.html -->
        <dependency>
            <groupId>org.springdoc</groupId>
            <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```

**引入依赖**

```plain
<!--openfeign-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

##### **6.4.2.3**** ****写YML**
```plain
server:
  port: 80

spring:
  application:
    name: cloud-consumer-openfeign-order
  ####Spring Cloud Consul for Service Discovery
  cloud:
    consul:
      host: localhost
      port: 8500
      discovery:
        prefer-ip-address: true #优先使用服务ip进行注册
        service-name: ${spring.application.name}
```

##### **6.4.2.4 主启动(****修改类名为****MainOpenFeign80)**
```plain
package com.atguigu.cloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.cloud.openfeign.EnableFeignClients;

/**
 * @auther zzyy
 * @create 2023-11-09 15:12
 */
@SpringBootApplication
@EnableDiscoveryClient //该注解用于向使用consul为注册中心时注册服务
@EnableFeignClients//启用feign客户端,定义服务+绑定接口，以声明式的方法优雅而简单的实现服务调用
public class MainOpenFeign80
{
    public static void main(String[] args)
    {
        SpringApplication.run(MainOpenFeign80.class,args);
    }
}
```

主启动类上面配置 @EnableFeignClients 表示开启OpenFeign功能并激活

@EnableFeignClients

##### **6.4.2.5 业务类**
**按照架构说明图进行编码准备**

订单模块要去调用支付模块，订单和支付两个微服务，需要通过Api接口解耦，一般不要在订单模块写非订单相关的业务，

自己的业务自己做+其它模块走FeignApi接口调用

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887187239-398ed891-45c6-4aa7-a1e5-08844cb1e41f.png)

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887189281-60b17497-b2ed-46d8-822b-f5ef3459049f.png)

**修改cloud-api-commons通用模块**

**引入openfeign依赖**

```plain
<!--openfeign-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

**新建服务接口PayFeignApi，头上配置@FeignClient注解**

@FeignClient

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887190666-a2c8dfb1-0f4e-4f3d-a595-e432ee504a03.png)

**参考微服务8001的Controller层，新建PayFeignApi接口**

```plain
package com.atguigu.cloud.apis;

import com.atguigu.cloud.entities.PayDTO;
import com.atguigu.cloud.resp.ResultData;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;

/**
 * @auther zzyy
 * @create 2023-11-09 15:29
 */
@FeignClient(value = "cloud-payment-service")
public interface PayFeignApi
{
    /**
     * 新增一条支付相关流水记录
     * @param payDTO
     * @return
     */
    @PostMapping("/pay/add")
    public ResultData addPay(@RequestBody PayDTO payDTO);

    /**
     * 按照主键记录查询支付流水信息
     * @param id
     * @return
     */
    @GetMapping("/pay/get/{id}")
    public ResultData getPayInfo(@PathVariable("id") Integer id);

    /**
     * openfeign天然支持负载均衡演示
     * @return
     */
    @GetMapping(value = "/pay/get/info")
    public String mylb();
}
```

**bug提醒一下**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887190719-71de21ed-29bc-40d1-a810-7835f9cd8270.png)

**拷贝之前的80工程进cloud-consumer-feign-order80，记得去掉部分代码和LoadBalancer不相关特性**

**修改Controller层的调用**

```plain
package com.atguigu.cloud.controller;

import com.atguigu.cloud.apis.PayFeignApi;
import com.atguigu.cloud.entities.PayDTO;
import com.atguigu.cloud.resp.ResultData;
import jakarta.annotation.Resource;
import lombok.extern.slf4j.Slf4j;
import org.springframework.web.bind.annotation.*;

/**
 * @auther zzyy
 * @create 2023-11-09 15:49
 */
@RestController
@Slf4j
public class OrderController
{
    @Resource
    private PayFeignApi payFeignApi;

    @PostMapping("/feign/pay/add")
    public ResultData addOrder(@RequestBody PayDTO payDTO)
    {
        System.out.println("第一步：模拟本地addOrder新增订单成功(省略sql操作)，第二步：再开启addPay支付微服务远程调用");
        ResultData resultData = payFeignApi.addPay(payDTO);
        return resultData;
    }

    @GetMapping("/feign/pay/get/{id}")
    public ResultData getPayInfo(@PathVariable("id") Integer id)
    {
        System.out.println("-------支付微服务远程调用，按照id查询订单支付流水信息");
        ResultData resultData = payFeignApi.getPayInfo(id);
        return resultData;
    }

    /**
     * openfeign天然支持负载均衡演示
     *
     * @return
     */
    @GetMapping(value = "/feign/pay/mylb")
    public String mylb()
    {
        return payFeignApi.mylb();
    }
}
```

#### 6.4.3 测试
**先启动Consul服务器**

**再启动微服务8001**

**再启动cloud-consumer-feign-order80**

**PostMan测试**

新增 [http://localhost/feign/pay/add](http://localhost/feign/pay/add)

查询 [http://localhost/feign/pay/get/1](http://localhost/feign/pay/get/1) 

**再启动微服务8002，测试看看O(∩_∩)O哈哈~**

[http://localhost/feign/pay/mylb](http://localhost/feign/pay/mylb)

OpenFeign默认集成了LoadBalancer

上述官网说明

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887190608-18036c53-561d-4f07-be97-dd8e4612a2b2.png)

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887191390-498653cb-7dde-4e65-b1ba-42396792c1d5.png)

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887192977-5f122a2d-e5e8-4fb8-9c24-3eda7705e15b.png)

#### 6.4.4 小总结
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887194589-d72d00d7-8cc0-41d5-b3e2-bc29efed6892.png)

### 6.5 OpenFeign高级特性
#### 6.5.1 OpenFeign超时控制
**本次OpenFeign的版本要注意，最新版和网络上你看到的配置不一样**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887194755-3d61d9bb-e093-4b8a-b443-f88534455123.png)

在Spring Cloud微服务架构中，大部分公司都是利用OpenFeign进行服务间的调用，而比较简单的业务使用默认配置是不会有多大问题的，但是如果是业务比较复杂，服务要进行比较繁杂的业务计算，那后台很有可能会出现Read Timeout这个异常，因此定制化配置超时时间就有必要了。

**超时设置，故意设置超时演示出错情况，自己使坏写bug**

**服务提供方cloud-provider-payment8001故意写暂停62秒钟程序**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887194886-25fcf410-23de-4611-95ec-fb451cdebf0f.png)

**服务调用方cloud-consumer-feign-order80写好捕捉超时异常**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887195107-cbaa44ca-3030-4eb0-9927-2e990fe19aee.png)

code

```plain
@GetMapping("/feign/pay/get/{id}")
public ResultData getPayInfo(@PathVariable("id") Integer id)
{
    System.out.println("-------支付微服务远程调用，按照id查询订单支付流水信息");
    ResultData resultData = null;
    try
    {
        System.out.println("调用开始-----:"+DateUtil.now());
        resultData = payFeignApi.getPayInfo(id);
    } catch (Exception e) {
        e.printStackTrace();
        System.out.println("调用结束-----:"+DateUtil.now());
        ResultData.fail(ReturnCodeEnum.RC500.getCode(),e.getMessage());
    }
    return resultData;
}
```

**测试**

[http://localhost/feign/pay/get/1](http://localhost/feign/pay/get/1)

错误页面

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887196580-b2787900-99f4-477b-a451-73373282da30.png)

**结论**

OpenFeign默认等待60秒钟，超过后报错

**官网解释+配置处理**

**两个关键参数**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887197622-d36b3fff-a324-44fe-9397-963b18725467.png)

默认OpenFeign客户端等待60秒钟，但是服务端处理超过规定时间会导致Feign客户端返回报错。

为了避免这样的情况，有时候我们需要设置Feign客户端的超时控制，默认60秒太长或者业务时间太短都不好

yml文件中开启配置：

connectTimeout       连接超时时间

readTimeout             请求处理超时时间

**超时配置参考官网要求**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887198607-0d784e85-2ac8-43ec-acc6-b7552d0b07c2.png)

**修改cloud-consumer-feign-order80，****YML****文件里需要开启OpenFeign客户端超时控制**

**官网出处**

[Spring Cloud OpenFeign](https://docs.spring.io/spring-cloud-openfeign/docs/current/reference/html/#spring-cloud-feign-overriding-defaults)

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887198488-e286a4f1-cfd3-4bf4-b857-ef7cb09eead4.png)

**全局配置**

**关键内容**

```plain
spring:
  cloud:
    openfeign:
      client:
        config:
          default:
            #连接超时时间
                      connectTimeout: 3000
            #读取超时时间
                     readTimeout: 3000
```

**all**

```plain
server:
  port: 80

spring:
  application:
    name: cloud-consumer-openfeign-order
  cloud:
    consul:
      host: localhost
      port: 8500
      discovery:
        prefer-ip-address: true #优先使用服务ip进行注册
                service-name: ${spring.application.name}
    openfeign:
      client:
        config:
          default:
            #连接超时时间
                        connectTimeout: 3000
            #读取超时时间
                        readTimeout: 3000
```

**3秒测试**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887199153-48155a9a-729e-4fc9-973f-dbefc20eb9d9.png)

**指定配置**

**单个服务配置超时时间（家庭作业）**

家庭作业

```plain
spring:
  cloud:
    openfeign:
      client:
        config:

          # default 设置的全局超时时间，指定服务名称可以设置单个服务的超时时间
          default:
             #连接超时时间
             connectTimeout: 4000
             #读取超时时间
             readTimeout: 4000

          # 为serviceC这个服务单独配置超时时间，单个配置的超时时间将会覆盖全局配置

          serviceC:
             #连接超时时间
             connectTimeout: 2000
             #读取超时时间
             readTimeout: 2000
```

**关键内容**

```plain
spring:
  cloud:
    openfeign:
      client:
        config:
          cloud-payment-service:
            #连接超时时间
                      connectTimeout: 5000
            #读取超时时间
                      readTimeout: 5000
```

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887200021-8afada62-c443-4c95-97ff-6feb27f0b725.png)**all**

```plain
server:
  port: 80

spring:
  application:
    name: cloud-consumer-openfeign-order
  ####Spring Cloud Consul for Service Discovery
  cloud:
    consul:
      host: localhost
      port: 8500
      discovery:
        prefer-ip-address: true #优先使用服务ip进行注册
                service-name: ${spring.application.name}
    openfeign:
      client:
        config:
         #default:
            #connectTimeout: 4000 #连接超时时间
                        #readTimeout: 4000 #读取超时时间
                   cloud-payment-service:
            connectTimeout: 8000 #连接超时时间
                        readTimeout: 8000 #读取超时时间
```

**5秒测试**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887200663-5e649ce0-ba9b-46d5-8f61-d10d831b4393.png)

#### 6.5.2 OpenFeign重试机制
**步骤**

**默认重试是关闭的，给了默认值**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887202095-6633e677-101b-4b30-95dc-263e7b466d3a.png)

**默认关闭重试机制，测试看看**

[http://localhost/feign/pay/get/1](http://localhost/feign/pay/get/1)

结果，只会调用一次后就结束

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887202203-ac1f1188-bbbb-4993-8ddf-4d6679715b8a.png)

**开启Retryer功能**

新增配置类FeignConfig并修改Retryer配置

```plain
package com.atguigu.cloud.config;

import feign.Retryer;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * @auther zzyy
 * @create 2023-11-10 11:09
 */
@Configuration
public class FeignConfig
{
    @Bean
    public Retryer myRetryer()
    {
        //return Retryer.NEVER_RETRY; //Feign默认配置是不走重试策略的

        //最大请求次数为3(1+2)，初始间隔时间为100ms，重试间最大间隔时间为1s
        return new Retryer.Default(100,1,3);
    }
}
```

结果，总体调用3次 

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887202905-234cd508-9150-4d2e-b37a-a113f3986d47.png)

3 = 1(default)+2

**补充一句**

如果你觉得效果不明显的同学，后续演示feign 日志功能的时候再演示，

目前控制台没有看到3次重试过程，只看到结果，_**正常的，正确的**_，是feign的日志打印问题

#### 6.5.3 OpenFeign默认HttpClient修改
**是什么**

OpenFeign中http client

如果不做特殊配置，OpenFeign默认使用JDK自带的HttpURLConnection发送HTTP请求，

由于默认HttpURLConnection没有连接池、性能和效率比较低，如果采用默认，性能上不是最牛B的，所以加到最大。

**替换之前，还是按照超时报错的案例**

```plain
@GetMapping("/feign/pay/get/{id}")
public ResultData getPayInfo(@PathVariable("id") Integer id)
{
    System.out.println("-------支付微服务远程调用，按照id查询订单支付流水信息");
    ResultData resultData = null;
    try
    {
        System.out.println("---调用开始: "+ DateUtil.now());
        resultData = payFeignApi.getPayInfo(id);
    } catch (Exception e) {
        e.printStackTrace();
        System.out.println("---调用结束: "+ DateUtil.now());
        return ResultData.fail(ReturnCodeEnum.RC500.getCode(),e.getMessage());
    }
    return resultData;
}
```

替换之前， 默认用的是什么

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887204066-bb6d3cbd-2d4f-406d-92f9-41c13bf5edad.png)

**Apache HttpClient 5替换 OpenFeign默认的HttpURLConnection**

**why**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887204560-3d78ea50-9e3b-4a97-9335-20675741320a.png)

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887205743-3b76e8c2-6d66-4e55-95ac-208bb5910694.png)

**修改微服务feign80，cloud-consumer-openfeign-order**

FeignConfig类里面将Retryer属性修改为默认

```plain
package com.atguigu.cloud.config;

import feign.Retryer;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * @auther zzyy
 * @create 2023-11-10 11:09
 */
@Configuration
public class FeignConfig
{
    @Bean
    public Retryer myRetryer()
    {
        return Retryer.NEVER_RETRY; //Feign默认配置是不走重试策略的
    }
}
```

POM修改

```plain
<!-- httpclient5-->
<dependency>
    <groupId>org.apache.httpcomponents.client5</groupId>
    <artifactId>httpclient5</artifactId>
    <version>5.3</version>
</dependency>
<!-- feign-hc5-->
<dependency>
    <groupId>io.github.openfeign</groupId>
    <artifactId>feign-hc5</artifactId>
    <version>13.1</version>
</dependency>
```

Apache HttpClient5  配置开启说明

```plain
#  Apache HttpClient5 配置开启
spring:
  cloud:
    openfeign:
      httpclient:
        hc5:
          enabled: true
```

YML 修改

```plain
server:
  port: 80

spring:
  application:
    name: cloud-consumer-openfeign-order
  ####Spring Cloud Consul for Service Discovery
  cloud:
    consul:
      host: localhost
      port: 8500
      discovery:
        prefer-ip-address: true #优先使用服务ip进行注册
        service-name: ${spring.application.name}
    openfeign:
      client:
        config:
          default:
            connectTimeout: 4000 #连接超时时间
                        readTimeout: 4000 #读取超时时间
            httpclient:
          hc5:
            enabled: true
          #cloud-payment-service:
            #connectTimeout: 4000 #连接超时时间
                        #readTimeout: 4000 #读取超时时间
```

**替换之前**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887205795-f59832fb-5be4-423f-abfd-79b1f3ad4cf9.png)

**替换之后**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887206318-28efe617-0b45-4800-9376-ec49deac7009.png)

#### 6.5.4 OpenFeign请求/响应压缩
**官网说明**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887206915-de7c51d6-d8d2-4add-95ec-0378d2fc3b3e.png)

**是什么**

**对请求和响应进行GZIP压缩**

Spring Cloud OpenFeign支持对请求和响应进行GZIP压缩，以减少通信过程中的性能损耗。

通过下面的两个参数设置，就能开启请求与相应的压缩功能：

spring.cloud.openfeign.compression.request.enabled=true

spring.cloud.openfeign.compression.response.enabled=true

**细粒度化设置**

对请求压缩做一些更细致的设置，比如下面的配置内容指定压缩的请求数据类型并设置了请求压缩的大小下限，

只有超过这个大小的请求才会进行压缩：

spring.cloud.openfeign.compression.request.enabled=true

spring.cloud.openfeign.compression.request.mime-types=text/xml,application/xml,application/json #触发压缩数据类型

spring.cloud.openfeign.compression.request.min-request-size=2048 #最小触发压缩的大小

**YML**

```plain
server:
  port: 80

spring:
  application:
    name: cloud-consumer-openfeign-order
  ####Spring Cloud Consul for Service Discovery
  cloud:
    consul:
      host: localhost
      port: 8500
      discovery:
        prefer-ip-address: true #优先使用服务ip进行注册
        service-name: ${spring.application.name}
    openfeign:
      client:
        config:
          default:
          #cloud-payment-service:
            #连接超时时间
                        connectTimeout: 4000
            #读取超时时间
                        readTimeout: 4000
      httpclient:
        hc5:
          enabled: true
      compression:
        request:
          enabled: true
          min-request-size: 2048 #最小触发压缩的大小
          mime-types: text/xml,application/xml,application/json #触发压缩数据类型
        response:
          enabled: true
```

**压缩效果测试在下一章节体现**

#### 6.5.5 OpenFeign日志打印功能
**日志打印功能**

**是什么**

Feign 提供了日志打印功能，我们可以通过配置来调整日志级别，

从而了解 Feign 中 Http 请求的细节，

说白了就是对Feign接口的调用情况进行监控和输出

**日志级别**

| NONE：默认的，不显示任何日志；<br/>BASIC：仅记录请求方法、URL、响应状态码及执行时间；<br/>HEADERS：除了 BASIC 中定义的信息之外，还有请求和响应的头信息；<br/>FULL：除了 HEADERS 中定义的信息之外，还有请求和响应的正文及元数据。 |
| :--- |


<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887207770-0dbd5d6e-a920-4d0f-b35e-fe55c0cef61b.png)

**配置日志bean**

```plain
package com.atguigu.cloud.config;

import feign.Logger;
import feign.Retryer;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * @auther zzyy
 * @create 2023-04-12 17:24
 */
@Configuration
public class FeignConfig
{
    @Bean
    public Retryer myRetryer()
    {
        return Retryer.NEVER_RETRY; //默认
    }

    @Bean
    Logger.Level feignLoggerLevel() {
        return Logger.Level.FULL;
    }
}
```

**YML文件里需要开启日志的Feign客户端**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887209920-76271fe8-7a2d-4407-b323-9d2cf7910a7c.png)

| 公式(三段)： **logging.level** +  含有@FeignClient注解的完整带包名的接口名+debug |
| :--- |


| _# feign_ _日志以什么级别监控哪个接口_   **logging**:    **level**:    **com**:   **atguigu**:   **cloud**:   **apis**:   **PayFeignApi**: debug  |
| :--- |


<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887209928-e3afc6fa-d3f5-491a-b27a-fd33e4ec2946.png)

**后台日志查看**

**带着压缩调用**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887210444-99e5d0ff-620f-4f9e-9ff6-9fe04dae0aa8.png)

**去掉压缩调用**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887211521-9842b3a1-20ed-4d1f-91b0-4dc69ae60fb9.png)

**补充实验，重试机制控制台看到3次过程**

**类FeignConfig**

```plain
package com.atguigu.cloud.config;

import feign.Logger;
import feign.Retryer;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * @auther zzyy
 * @create 2023-11-10 11:09
 */
@Configuration
public class FeignConfig
{
    @Bean
    public Retryer myRetryer()
    {
        //最大请求次数为3(1+2)，初始间隔时间为100ms，重试间最大间隔时间为1s
        return new Retryer.Default(100,1,3);
    }
    @Bean
    Logger.Level feignLoggerLevel() {
        return Logger.Level.FULL;
    }
}
```

**YML（看到效果改为2秒）**

```plain
server:
  port: 80

spring:
  application:
    name: cloud-consumer-openfeign-order
  ####Spring Cloud Consul for Service Discovery
  cloud:
    consul:
      host: localhost
      port: 8500
      discovery:
        prefer-ip-address: true #优先使用服务ip进行注册
        service-name: ${spring.application.name}
    openfeign:
      client:
        config:
          default:
          #cloud-payment-service:
            #连接超时时间
                        connectTimeout: 2000
            #读取超时时间
                        readTimeout: 2000
      httpclient:
        hc5:
          enabled: true
      compression:
        request:
          enabled: true
          min-request-size: 2048
          mime-types: text/xml,application/xml,application/json
        response:
          enabled: true

# feign日志以什么级别监控哪个接口
logging:
  level:
    com:
      atguigu:
        cloud:
          apis:
            PayFeignApi: debug
```

**测试地址**

[http://localhost/feign/pay/get/1](http://localhost/feign/pay/get/1)

**控制台3次重试触发效果的过程**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887213028-56bb0610-6dd8-493c-baaa-88685920f2c2.png)

**本节内容最后的YML**

```plain
server:
  port: 80

spring:
  application:
    name: cloud-consumer-openfeign-order
  ####Spring Cloud Consul for Service Discovery
  cloud:
    consul:
      host: localhost
      port: 8500
      discovery:
        prefer-ip-address: true #优先使用服务ip进行注册
        service-name: ${spring.application.name}
    openfeign:
      client:
        config:
          default:
            connectTimeout: 2000 #连接超时时间
                        readTimeout: 2000 #读取超时时间
           httpclient:
        hc5:
          enabled: true
      compression:
         request:
           enabled: true
           min-request-size: 2048
           mime-types: text/xml,application/xml,application/json
         response:
           enabled: true
          #cloud-payment-service:
            #connectTimeout: 4000 #连接超时时间
            #readTimeout: 4000 #读取超时时间

# feign日志以什么级别监控哪个接口
logging:
  level:
    com:
      atguigu:
        cloud:
          apis:
            PayFeignApi: debug
```

### **6.6 OpenFeign和Sentinel集成实现fallback服务降级**
## 7 CircuitBreaker断路器
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887259823-69d3d4ba-4ae7-47d4-8e95-56b032881b98.png)

### 7.1 Hystrix目前也进入维护模式
#### 7.1.1 是什么
Hystrix是一个用于处理分布式系统的延迟和容错的开源库，在分布式系统里，许多依赖不可避免的会调用失败，比如超时、异常等，Hystrix能够保证在一个依赖出问题的情况下，不会导致整体服务失败，避免级联故障，以提高分布式系统的弹性。

了解一下即可，2024年了不再使用Hystrix

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887259618-23eccb86-46e8-41ce-8cb4-d8f8210c8ea2.png)

#### 7.1.2 Hystrix官宣，停更进维
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887259454-bc9547e2-c338-4394-90f2-18ade43e6377.png)

#### 7.1.3 Hystrix未来替换方案
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887259080-a43e0603-625e-4e20-9efa-6ea6b6bb5b67.png)

**Resilience4j **

### 7.2 概述
#### 7.2.1 2023年影响极大的真实生产故障
**语雀崩了(2023.10.23)**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887260004-cf752863-b6e9-4964-9fc0-0bbed9a64b97.png)

**阿里系大部分产品(2023.11.12)**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887262670-f7b3f19b-3c5e-4ae2-98a1-8710f7ac9528.png)

**阿里云产品控制台**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887264233-6283b894-6649-4c8e-9280-18a4979f115b.png)

#### 7.2.2 分布式系统面临的问题
分布式系统面临的问题

复杂分布式体系结构中的应用程序有数十个依赖关系，每个依赖关系在某些时候将不可避免地失败。

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887265996-fa386677-38c7-4d7b-9839-29bcbe4c4d31.png)

服务雪崩

多个微服务之间调用的时候，假设微服务A调用微服务B和微服务C，微服务B和微服务C又调用其它的微服务，这就是所谓的“扇出”。如果扇出的链路上某个微服务的调用响应时间过长或者不可用，对微服务A的调用就会占用越来越多的系统资源，进而引起系统崩溃，所谓的“雪崩效应”.

对于高流量的应用来说，单一的后端依赖可能会导致所有服务器上的所有资源都在几秒钟内饱和。比失败更糟糕的是，这些应用程序还可能导致服务之间的延迟增加，备份队列，线程和其他系统资源紧张，导致整个系统发生更多的级联故障。这些都表示需要对故障和延迟进行隔离和管理，以便单个依赖关系的失败，不能取消整个应用程序或系统。

所以，通常当你发现一个模块下的某个实例失败后，这时候这个模块依然还会接收流量，然后这个有问题的模块还调用了其他的模块，这样就会发生级联故障，或者叫雪崩。

#### 7.2.3 我们的诉求
问题：

禁止服务雪崩故障

解决： 

- 有问题的节点，快速熔断（快速返回失败处理或者返回默认兜底数据【服务降级】）。

“断路器”本身是一种开关装置，当某个服务单元发生故障之后，通过断路器的故障监控（类似熔断保险丝），向调用方返回一个符合预期的、可处理的备选响应(FallBack)，而不是长时间的等待或者抛出调用方无法处理的异常，这样就保证了服务调用方的线程不会被长时间、不必要地占用，从而避免了故障在分布式系统中的蔓延，乃至雪崩。

一句话，**出故障了“保险丝”跳闸，别把整个家给烧了，****😄**

#### 7.2.4 如何搞定上述问题，避免整个系统大面积故障
**给我搞定**

**服务熔断**

类比保险丝，保险丝闭合状态(CLOSE)可以正常使用，当达到最大服务访问后，直接拒绝访问跳闸限电(OPEN)，此刻调用方会接受服务降级的处理并返回友好兜底提示

就是家里保险丝，从闭合CLOSE供电状态→跳闸OPEN打开状态

**服务降级**

服务器忙，请稍后再试。

不让客户端等待并立刻返回一个友好提示，fallback

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887264827-64be0239-ed6e-420d-a3d2-9ea9522f6a5b.png)

**服务限流**

秒杀高并发等操作，严禁一窝蜂的过来拥挤，大家排队，一秒钟N个，有序进行

**服务限时**

**服务预热**

**接近实时的监控**

**兜底的处理动作**

**。。。。。。**

**NOW**

我们用什么替代？

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887264791-039a06be-1c85-4cb1-b599-20a6b49e21ef.png)

Spring Cloud Circuit Breaker

### 7.3 Circuit Breaker是什么
**官网**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887266158-24fb9538-5f70-4e7a-9e9a-8cbf9fab5722.png)

[Spring Cloud Circuit Breaker](https://spring.io/projects/spring-cloud-circuitbreaker#overview)

**实现原理**

CircuitBreaker的目的是保护分布式系统免受故障和异常，提高系统的可用性和健壮性。

当一个组件或服务出现故障时，CircuitBreaker会迅速切换到开放OPEN状态(保险丝跳闸断电)，阻止请求发送到该组件或服务从而避免更多的请求发送到该组件或服务。这可以减少对该组件或服务的负载，防止该组件或服务进一步崩溃，并使整个系统能够继续正常运行。同时，CircuitBreaker还可以提高系统的可用性和健壮性，因为它可以在分布式系统的各个组件之间自动切换，从而避免单点故障的问题。

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887268764-382b7746-8e38-450d-9c32-6e127be6edc8.png)

**一句话**

Circuit Breaker只是一套规范和接口，落地实现者是Resilience4J

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887269345-7f39af66-073d-4929-825a-7f715aa99372.png)

### 7.4 Resilience4J
**是什么**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887269242-19268c14-f5dc-4562-bd98-0be07c42584c.png)

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887270409-2a48c239-5ef4-4f83-b8c3-8bd3b40986c9.png)

[https://github.com/resilience4j/resilience4j#1-introduction](https://github.com/resilience4j/resilience4j#1-introduction)

**能干嘛**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887270437-b535010e-5579-44ec-b4dd-ae2acc3c47ff.png)

[https://github.com/resilience4j/resilience4j#3-overview](https://github.com/resilience4j/resilience4j#3-overview)

**怎么玩**

**官网**

[CircuitBreaker](https://resilience4j.readme.io/docs/circuitbreaker)

**中文手册**

[https://github.com/lmhmhl/Resilience4j-Guides-Chinese/blob/main/index.md](https://github.com/lmhmhl/Resilience4j-Guides-Chinese/blob/main/index.md)

### 7.5 案例实战
#### 7.5.1 熔断(CircuitBreaker)(服务熔断+服务降级)
##### 7.5.1.1 断路器3大状态
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887273697-ea01d181-633e-4f0e-acd9-2a52a5b6fadd.png)

##### 7.5.1.2 断路器3大状态之间的转换
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887275541-3da3f35a-5c69-4bae-9719-af1dc1eb83e4.png)

##### 7.5.1.3 断路器所有配置参数参考
**英文**

[https://resilience4j.readme.io/docs/circuitbreaker#create-and-configure-a-circuitbreaker](https://resilience4j.readme.io/docs/circuitbreaker#create-and-configure-a-circuitbreaker)

**中文手册**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887275397-7fc2a9a5-6a5c-4ca5-a5fc-eba15c3696d8.png)

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887276102-e24a4fcd-fff5-4092-b57e-45a426be673b.png)

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887275840-a9381d6e-92f3-4b56-bb64-d229e5bea80d.png)

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887277087-a1ce0cd9-67c7-42b7-be2d-7926921c0cf5.png)

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887279675-760befba-5978-46f7-99f3-0ab6d985bd37.png)


**默认CircuitBreaker.java配置类**

io.github.resilience4j.circuitbreaker.CircuitBreakerConfig

**中文手册精简版**

| **failure-rate-threshold ** | **以百分比配置失败率峰值** |
| :--- | :--- |
| **sliding-window-type ** | **断路器的滑动窗口期类型   ****可以基于“次数”（COUNT_BASED）或者“时间”（TIME_BASED）进行熔断，默认是COUNT_BASED。** |
| **sliding-window-size** | **若COUNT_BASED，则10次调用中有50%失败（即5次）打开熔断断路器；**<br/>**若为TIME_BASED则，此时还有额外的两个设置属性，含义为：在N秒内（sliding-window-size）100%（slow-call-rate-threshold）的请求超过N秒（slow-call-duration-threshold）打开断路器。** |
| **slowCallRateThreshold** | **以百分比的方式配置，断路器把调用时间大于slowCallDurationThreshold的调用视为慢调用，当慢调用比例大于等于峰值时，断路器开启，并进入服务降级。** |
| **slowCallDurationThreshold** | **配置调用时间的峰值，高于该峰值的视为慢调用。** |
| **permitted-number-of-calls-in-half-open-state** | **运行断路器在HALF_OPEN状态下时进行N次调用，如果故障或慢速调用仍然高于阈值，断路器再次进入打开状态。** |
| **minimum-number-of-calls** | **在每个滑动窗口期样本数，配置断路器计算错误率或者慢调用率的最小调用数。比如设置为5意味着，在计算故障率之前，必须至少调用5次。如果只记录了4次，即使4次都失败了，断路器也不会进入到打开状态。** |
| **wait-duration-in-open-state** | **从OPEN到HALF_OPEN状态需要等待的时间** |


##### 7.5.1.4 熔断+降级案例需求说明
6次访问中当执行方法的失败率达到50%时CircuitBreaker将进入开启OPEN状态(保险丝跳闸断电)拒绝所有请求。  
 等待5秒后，CircuitBreaker 将自动从开启OPEN状态过渡到半开HALF_OPEN状态，允许一些请求通过以测试服务是否恢复正常。  
如还是异常CircuitBreaker 将重新进入开启OPEN状态；如正常将进入关闭CLOSE闭合状态恢复正常处理请求。

具体时间和频次等属性见具体实际案例，这里只是作为case举例讲解，最下面笔记面试题概览，闲聊大厂面试   

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887280583-60c6a637-f3c5-43f4-948e-9951c382310c.png)

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887283079-58805124-653c-4db3-abad-dbdf4af343c1.png)

##### **7.5.1.5 干，按照COUNT_BASED(计数的滑动窗口)**
**修改cloud-provider-payment8001**

新建PayCircuitController

```plain
/**
 * @auther zzyy
 * @create 2023-11-13 14:55
 */
@RestController
public class PayCircuitController
{
    //=========Resilience4j CircuitBreaker 的例子
    @GetMapping(value = "/pay/circuit/{id}")
    public String myCircuit(@PathVariable("id") Integer id)
    {
        if(id == -4) throw new RuntimeException("----circuit id 不能负数");
        if(id == 9999){
            try { TimeUnit.SECONDS.sleep(5); } catch (InterruptedException e) { e.printStackTrace(); }
        }
        return "Hello, circuit! inputId:  "+id+" \t " + IdUtil.simpleUUID();
    }
}
```

**修改PayFeignApi接口**

```plain
/**
 * @auther zzyy
 * @create 2023-11-09 15:29
 */
@FeignClient(value = "cloud-payment-service")
public interface PayFeignApi
{
    /**
     * 新增一条支付相关流水记录
     * @param payDTO
     * @return
     */
    @PostMapping("/pay/add")
    public ResultData addPay(@RequestBody PayDTO payDTO);

    /**
     * 按照主键记录查询支付流水信息
     * @param id
     * @return
     */
    @GetMapping("/pay/get/{id}")
    public ResultData getPayInfo(@PathVariable("id") Integer id);

    /**
     * openfeign天然支持负载均衡演示
     * @return
     */
    @GetMapping(value = "/pay/get/info")
    public String mylb();

    /**
     * Resilience4j CircuitBreaker 的例子
     * @param id
     * @return
     */
    @GetMapping(value = "/pay/circuit/{id}")
    public String myCircuit(@PathVariable("id") Integer id);
}
```

**修改cloud-consumer-feign-order80**

**改POM**

```plain
<!--resilience4j-circuitbreaker-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-circuitbreaker-resilience4j</artifactId>
</dependency>
<!-- 由于断路保护等需要AOP实现，所以必须导入AOP包 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```

**写YML**

```plain
server:
  port: 80

spring:
  application:
    name: cloud-consumer-openfeign-order
  ####Spring Cloud Consul for Service Discovery
  cloud:
    consul:
      host: localhost
      port: 8500
      discovery:
        prefer-ip-address: true #优先使用服务ip进行注册
        service-name: ${spring.application.name}
    openfeign:
      client:
        config:
          default:
            #cloud-payment-service:
            #连接超时时间，为避免演示出错，讲解完本次内容后设置为20秒
            connectTimeout: 20000
            #读取超时时间，为避免演示出错，讲解完本次内容后设置为20秒
            readTimeout: 20000
            #开启httpclient5
      httpclient:
        hc5:
          enabled: true
          #开启压缩特性
            compression:
        request:
          enabled: true
          min-request-size: 2048
          mime-types: text/xml,application/xml,application/json
        response:
          enabled: true
       # 开启circuitbreaker和分组激活 spring.cloud.openfeign.circuitbreaker.enabled
            circuitbreaker:
        enabled: true
        group:
          enabled: true #没开分组永远不用分组的配置。精确优先、分组次之(开了分组)、默认最后


# feign日志以什么级别监控哪个接口
logging:
  level:
    com:
      atguigu:
        cloud:
          apis:
            PayFeignApi: debug

# Resilience4j CircuitBreaker 按照次数：COUNT_BASED 的例子
#  6次访问中当执行方法的失败率达到50%时CircuitBreaker将进入开启OPEN状态(保险丝跳闸断电)拒绝所有请求。
#  等待5秒后，CircuitBreaker 将自动从开启OPEN状态过渡到半开HALF_OPEN状态，允许一些请求通过以测试服务是否恢复正常。
#  如还是异常CircuitBreaker 将重新进入开启OPEN状态；如正常将进入关闭CLOSE闭合状态恢复正常处理请求。
resilience4j:
  circuitbreaker:
    configs:
      default:
        failureRateThreshold: 50 #设置50%的调用失败时打开断路器，超过失败请求百分⽐CircuitBreaker变为OPEN状态。
                slidingWindowType: COUNT_BASED # 滑动窗口的类型
                slidingWindowSize: 6 #滑动窗⼝的⼤⼩配置COUNT_BASED表示6个请求，配置TIME_BASED表示6秒
                minimumNumberOfCalls: 6 #断路器计算失败率或慢调用率之前所需的最小样本(每个滑动窗口周期)。如果minimumNumberOfCalls为10，则必须最少记录10个样本，然后才能计算失败率。如果只记录了9次调用，即使所有9次调用都失败，断路器也不会开启。
                automaticTransitionFromOpenToHalfOpenEnabled: true # 是否启用自动从开启状态过渡到半开状态，默认值为true。如果启用，CircuitBreaker将自动从开启状态过渡到半开状态，并允许一些请求通过以测试服务是否恢复正常
                waitDurationInOpenState: 5s #从OPEN到HALF_OPEN状态需要等待的时间
                permittedNumberOfCallsInHalfOpenState: 2 #半开状态允许的最大请求数，默认值为10。在半开状态下，CircuitBreaker将允许最多permittedNumberOfCallsInHalfOpenState个请求通过，如果其中有任何一个请求失败，CircuitBreaker将重新进入开启状态。
                recordExceptions:
          - java.lang.Exception
    instances:
      cloud-payment-service:
        baseConfig: default
```

**新建OrderCircuitController**

```plain
package com.atguigu.cloud.controller;

import com.atguigu.cloud.apis.PayFeignApi;
import io.github.resilience4j.circuitbreaker.annotation.CircuitBreaker;
import jakarta.annotation.Resource;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;

/**
 * @auther zzyy
 * @create 2023-11-13 14:54
 * Resilience4j CircuitBreaker 的例子
 */
@RestController
public class OrderCircuitController
{
    @Resource
    private PayFeignApi payFeignApi;

    @GetMapping(value = "/feign/pay/circuit/{id}")
    @CircuitBreaker(name = "cloud-payment-service", fallbackMethod = "myCircuitFallback")
    public String myCircuitBreaker(@PathVariable("id") Integer id)
    {
        return payFeignApi.myCircuit(id);
    }
    //myCircuitFallback就是服务降级后的兜底处理方法
        public String myCircuitFallback(Integer id,Throwable t) {
        // 这里是容错处理逻辑，返回备用结果
        return "myCircuitFallback，系统繁忙，请稍后再试-----/(ㄒoㄒ)/~~";
    }
}
```

@CircuitBreaker

系统繁忙，请稍后再试。

不让调用者等待并立刻返回一个友好提示，fallback

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887281851-65ba4ae6-a052-4c83-b8bd-da4da216b81b.png)

**测试(按照错误次数达到多少后开启断路)**

**自测cloud-consumer-feign-order80**

**查看YML**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887282764-82423ba5-5936-4884-aa51-900a909dd1bf.png)

**正确**

[http://localhost/feign/pay/circuit/11](http://localhost/feign/pay/circuit/11)

**错误**

[http://localhost/feign/pay/circuit/-4](http://localhost/feign/pay/circuit/-4)

**一次error一次OK，trytry看看**

50%错误后触发熔断并给出服务降级，告知调用者服务不可用

此时就算是输入正确的访问地址也无法调用服务(我明明是正确的也不让用/(ㄒoㄒ)/~~)，它还在断路中(OPEN状态)，一会儿过渡到半开并继续正确地址访问，慢慢切换回CLOSE状态，可以正常访问了链路恢复

**多次故意填写错误值（负4）**

多次故意填写错误值(负4)，然后慢慢填写正确值(正整数11)，发现刚开始不满足条件，就算是正确的访问地址也不能进行

##### **7.5.1.6 干，按照TIME_BASED(时间的滑动窗口)**
**基于时间的滑动窗口**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887284831-36cbb8d1-591a-472e-aff6-9e99150c2e5d.png)

**修改cloud-consumer-feign-order80**

写YML

```plain
server:
  port: 80

spring:
  application:
    name: cloud-consumer-openfeign-order
  ####Spring Cloud Consul for Service Discovery
  cloud:
    consul:
      host: localhost
      port: 8500
      discovery:
        prefer-ip-address: true #优先使用服务ip进行注册
        service-name: ${spring.application.name}
    openfeign:
      client:
        config:
          default:
            #cloud-payment-service:
            #连接超时时间，为避免演示出错，讲解完本次内容后设置为20秒
            connectTimeout: 20000
            #读取超时时间，为避免演示出错，讲解完本次内容后设置为20秒
            readTimeout: 20000
            #开启httpclient5
      httpclient:
        hc5:
          enabled: true
          #开启压缩特性
      compression:
        request:
          enabled: true
          min-request-size: 2048
          mime-types: text/xml,application/xml,application/json
        response:
          enabled: true
      #开启circuitbreaker和分组激活
      circuitbreaker:
        enabled: true
        group:
          enabled: true #没开分组永远不用分组的配置。精确优先、分组次之(开了分组)、默认最后


# feign日志以什么级别监控哪个接口
logging:
  level:
    com:
      atguigu:
        cloud:
          apis:
            PayFeignApi: debug

# Resilience4j CircuitBreaker 按照时间：TIME_BASED 的例子
resilience4j:
  timelimiter:
    configs:
      default:
        timeout-duration: 10s #神坑的位置，timelimiter 默认限制远程1s，超于1s就超时异常，配置了降级，就走降级逻辑
  circuitbreaker:
    configs:
      default:
        failureRateThreshold: 50 #设置50%的调用失败时打开断路器，超过失败请求百分⽐CircuitBreaker变为OPEN状态。
        slowCallDurationThreshold: 2s #慢调用时间阈值，高于这个阈值的视为慢调用并增加慢调用比例。
        slowCallRateThreshold: 30 #慢调用百分比峰值，断路器把调用时间⼤于slowCallDurationThreshold，视为慢调用，当慢调用比例高于阈值，断路器打开，并开启服务降级
        slidingWindowType: TIME_BASED # 滑动窗口的类型
        slidingWindowSize: 2 #滑动窗口的大小配置，配置TIME_BASED表示2秒
        minimumNumberOfCalls: 2 #断路器计算失败率或慢调用率之前所需的最小样本(每个滑动窗口周期)。
        permittedNumberOfCallsInHalfOpenState: 2 #半开状态允许的最大请求数，默认值为10。
        waitDurationInOpenState: 5s #从OPEN到HALF_OPEN状态需要等待的时间
        recordExceptions:
          - java.lang.Exception
    instances:
      cloud-payment-service:
        baseConfig: default
```

**为避免影响实验效果，记得关闭FeignConfig自己写的重试3次**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887284601-efc38fac-c570-43ef-b73d-9725175fa5ba.png)

**测试(慢查询)**

**一次超时，一次正常访问，同时进行**

[http://localhost/feign/pay/circuit/9999](http://localhost/feign/pay/circuit/9999)        故意超时，将会单独报错

[http://localhost/feign/pay/circuit/11](http://localhost/feign/pay/circuit/11)        可以访问，我是正常的

**第1~4个超时，整多一点干4个，一次正常访问，同时进行**

[http://localhost/feign/pay/circuit/9999](http://localhost/feign/pay/circuit/9999)

**正常访问也受到了牵连，因为服务熔断不能访问了**

[http://localhost/feign/pay/circuit/11](http://localhost/feign/pay/circuit/11)

**运气好的话，可以看到全线崩，刺激。**

##### 7.5.1.7 小总结
**断路器开启或者关闭的条件**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887286653-e0615cf5-62b2-4719-a8d3-5a4a40279021.png)

当满足一定的峰值和失败率达到一定条件后，断路器将会进入OPEN状态(保险丝跳闸)，服务熔断

当OPEN的时候，所有请求都不会调用主业务逻辑方法，而是直接走fallbackmetnod兜底背锅方法，服务降级

一段时间之后，这个时候断路器会从OPEN进入到HALF_OPEN半开状态，会放几个请求过去探探链路是否通？

如成功，断路器会关闭CLOSE(类似保险丝闭合，恢复可用)；

如失败，继续开启。重复上述

**个人建议不要混合用，****推荐按照调用次数count_based****，一家之言仅供参考**

#### 7.5.2 隔离(BulkHead)
##### **7.5.2.1 官网**
[https://resilience4j.readme.io/docs/bulkhead](https://resilience4j.readme.io/docs/bulkhead)

中文

[Resilience4j-Guides-Chinese/core-modules/bulkhead.md at main · lmhmhl/Resilience4j-Guides-Chinese · GitHub](https://github.com/lmhmhl/Resilience4j-Guides-Chinese/blob/main/core-modules/bulkhead.md)

##### **7.5.2.2 是什么**
bulkhead(船的)舱壁/(飞机的)隔板

隔板来自造船行业，床仓内部一般会分成很多小隔舱，一旦一个隔舱漏水因为隔板的存在而不至于影响其它隔舱和整体船。

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887287395-1105cd76-60d8-45c3-8a6e-14851acca1ac.png)

限并发

##### **7.5.2.3 能干吗**
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887288027-630afe3e-95a6-4db8-a612-9f9c2c1737e0.png)

**依赖隔离&负载保护：**用来限制对于下游服务的最大并发数量的限制

##### **7.5.2.4 **Resilience4j提供了如下两种隔离的实现方式，可以限制并发执行的数量
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887289874-51c37bc3-6f31-4aaf-bfff-b6452f778fa5.png)

##### **7.5.2.5 **实现SemaphoreBulkhead(信号量舱壁)
**概述**

基本上就是我们JUC信号灯内容的同样思想

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887289865-4d52208e-dda6-4f49-ab04-b9095d9e5b62.png)

信号量舱壁（SemaphoreBulkhead）原理

当信号量有空闲时，进入系统的请求会直接获取信号量并开始业务处理。

当信号量全被占用时，接下来的请求将会进入阻塞状态，SemaphoreBulkhead提供了一个阻塞计时器，

如果阻塞状态的请求在阻塞计时内无法获取到信号量则系统会拒绝这些请求。

若请求在阻塞计时内获取到了信号量，那将直接获取信号量并执行相应的业务处理。

**源码分析**

io.github.resilience4j.bulkhead.internal.SemaphoreBulkhead

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887291006-dc9233ef-3457-45c3-a42d-8334a5095987.png)

**cloud-provider-payment8001支付微服务 修改PayCircuitController**

```plain
//=========Resilience4j bulkhead 的例子
@GetMapping(value = "/pay/bulkhead/{id}")
public String myBulkhead(@PathVariable("id") Integer id)
{
    if(id == -4) throw new RuntimeException("----bulkhead id 不能-4");

    if(id == 9999)
    {
        try { TimeUnit.SECONDS.sleep(5); } catch (InterruptedException e) { e.printStackTrace(); }
    }

    return "Hello, bulkhead! inputId:  "+id+" \t " + IdUtil.simpleUUID();
}
```

**PayFeignApi接口新增舱壁api方法**

```plain
/**
 * Resilience4j Bulkhead 的例子
 * @param id
 * @return
 */
@GetMapping(value = "/pay/bulkhead/{id}")
public String myBulkhead(@PathVariable("id") Integer id);
```

**修改cloud-consumer-feign-order80**

**POM**

```plain
<!--resilience4j-bulkhead-->
<dependency>
    <groupId>io.github.resilience4j</groupId>
    <artifactId>resilience4j-bulkhead</artifactId>
</dependency>
```

**YML**

**示例**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887292546-e7f69b36-25e8-4259-812d-5e4df2634ab1.png)

**内容**

```plain
server:
  port: 80

spring:
  application:
    name: cloud-consumer-openfeign-order
  ####Spring Cloud Consul for Service Discovery
  cloud:
    consul:
      host: localhost
      port: 8500
      discovery:
        prefer-ip-address: true #优先使用服务ip进行注册
        service-name: ${spring.application.name}
    openfeign:
      client:
        config:
          default:
          #cloud-payment-service:
            #连接超时时间，为避免演示出错，讲解完本次内容后设置为20秒
            connectTimeout: 20000
            #读取超时时间，为避免演示出错，讲解完本次内容后设置为20秒
            readTimeout: 20000
            #开启httpclient5
      httpclient:
        hc5:
          enabled: true
          #开启压缩特性
      compression:
        request:
          enabled: true
          min-request-size: 2048
          mime-types: text/xml,application/xml,application/json
        response:
          enabled: true
      #开启circuitbreaker和分组激活
      circuitbreaker:
        enabled: true
        group:
          enabled: true #没开分组永远不用分组的配置。精确优先、分组次之(开了分组)、默认最后


# feign日志以什么级别监控哪个接口
logging:
  level:
    com:
      atguigu:
        cloud:
          apis:
            PayFeignApi: debug

####resilience4j bulkhead 的例子
resilience4j:
  bulkhead:
    configs:
      default:
        maxConcurrentCalls: 2 # 隔离允许并发线程执行的最大数量
                maxWaitDuration: 1s # 当达到并发调用数量时，新的线程的阻塞时间，我只愿意等待1秒，过时不候进舱壁兜底fallback
    instances:
      cloud-payment-service:
        baseConfig: default
  timelimiter:
    configs:
      default:
        timeout-duration: 20s #神坑的位置，timelimiter 默认限制远程1s，超于1s就超时异常，配置了降级，就走降级逻辑
```

**业务类**

**OrderCircuitController**

```plain
/**
 *(船的)舱壁,隔离
 * @param id
 * @return
 */
@GetMapping(value = "/feign/pay/bulkhead/{id}")
@Bulkhead(name = "cloud-payment-service",fallbackMethod = "myBulkheadFallback",type = Bulkhead.Type.SEMAPHORE)
public String myBulkhead(@PathVariable("id") Integer id)
{
    return payFeignApi.myBulkhead(id);
}
public String myBulkheadFallback(Throwable t)
{
    return "myBulkheadFallback，隔板超出最大数量限制，系统繁忙，请稍后再试-----/(ㄒoㄒ)/~~";
}
```

@Bulkhead 

Bulkhead.Type.SEMAPHORE

**测试**

**步骤**

浏览器新打开2个窗口，各点一次，分别点击http://localhost/feign/pay/bulkhead/9999

每个请求调用需要耗时5秒，2个线程瞬间达到配置过的最大并发数2

此时第3个请求正常的请求访问，http://localhost/feign/pay/bulkhead/3

直接被舱壁限制隔离了，碰不到8001

等其中一个窗口停止了，再去正常访问，并发数小于2 了，可以OK

[http://localhost/feign/pay/bulkhead/9999](http://localhost/feign/pay/bulkhead/9999)

[http://localhost/feign/pay/bulkhead/3](http://localhost/feign/pay/bulkhead/3)

**结果**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887292995-7b7cd47d-f77c-4b89-8932-cfecd51385b3.png)

可以看到因为本案例并发线程数为2（maxConcurrentCalls: 2），只让2个线程进入执行，

其他请求降直接降级。

##### **7.5.2.6 **实现FixedThreadPoolBulkhead(固定线程池舱壁)
**概述**

基本上就是我们JUC-线程池内容的同样思想

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887294221-55acf1f3-71f2-4bac-9a0e-0dc76e4f1772.png)

固定线程池舱壁（FixedThreadPoolBulkhead）

FixedThreadPoolBulkhead的功能与SemaphoreBulkhead一样也是**用于限制并发执行的次数**的，但是二者的实现原理存在差别而且表现效果也存在细微的差别。FixedThreadPoolBulkhead使用一个固定线程池和一个等待队列来实现舱壁。

当线程池中存在空闲时，则此时进入系统的请求将直接进入线程池开启新线程或使用空闲线程来处理请求。

当线程池中无空闲时时，接下来的请求将进入等待队列，

若等待队列仍然无剩余空间时接下来的请求将直接被拒绝，

在队列中的请求等待线程池出现空闲时，将进入线程池进行业务处理。

另外：ThreadPoolBulkhead只对CompletableFuture方法有效，所以我们必创建返回CompletableFuture类型的方法

**源码分析**

**io.github.resilience4j.bulkhead.internal.FixedThreadPoolBulkhead**

**底子就是JUC里面的线程池ThreadPoolExecutor**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887294449-5b942fff-198d-4027-90a2-79fd0ed19800.png)

**submit进线程池返回CompletableFuture<>**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887295496-5e2ecdef-7edc-4a56-9eda-797fdf63ee45.png)

**修改cloud-consumer-feign-order80**

**POM**

```plain
<!--resilience4j-bulkhead-->
<dependency>
    <groupId>io.github.resilience4j</groupId>
    <artifactId>resilience4j-bulkhead</artifactId>
</dependency>
```

**YML**

**示例**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887298514-58bf0b37-d959-4cea-b296-e55344007482.png)

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887298495-d8a2819f-37ef-442d-90d8-741fbd031861.png)

**内容**

```plain
server:
  port: 80

spring:
  application:
    name: cloud-consumer-openfeign-order
  ####Spring Cloud Consul for Service Discovery
  cloud:
    consul:
      host: localhost
      port: 8500
      discovery:
        prefer-ip-address: true #优先使用服务ip进行注册
        service-name: ${spring.application.name}
    openfeign:
      client:
        config:
          default:
            #cloud-payment-service:
            #连接超时时间，为避免演示出错，讲解完本次内容后设置为20秒
            connectTimeout: 20000
            #读取超时时间，为避免演示出错，讲解完本次内容后设置为20秒
            readTimeout: 20000
            #开启httpclient5
      httpclient:
        hc5:
          enabled: true
          #开启压缩特性
      compression:
        request:
          enabled: true
          min-request-size: 2048
          mime-types: text/xml,application/xml,application/json
        response:
          enabled: true
      #开启circuitbreaker和分组激活
      circuitbreaker:
        enabled: true
#        group:
#          enabled: true 
# 演示Bulkhead.Type.THREADPOOL时spring.cloud.openfeign.circuitbreaker.group.enabled 
# 设为false新启线程和原来主线程脱离了。



# feign日志以什么级别监控哪个接口
logging:
  level:
    com:
      atguigu:
        cloud:
          apis:
            PayFeignApi: debug

####resilience4j bulkhead -THREADPOOL的例子
resilience4j:
  timelimiter:
    configs:
      default:
        timeout-duration: 10s #timelimiter默认限制远程1s，超过报错不好演示效果所以加上10秒
  thread-pool-bulkhead:
    configs:
      default:
        core-thread-pool-size: 1
        max-thread-pool-size: 1
        queue-capacity: 1
    instances:
      cloud-payment-service:
        baseConfig: default
# spring.cloud.openfeign.circuitbreaker.group.enabled 请设置为false 新启线程和原来主线程脱离
```

**上述内容解释**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887299231-7ce17fa7-7686-46f6-8f33-20f746eb1c47.png)

**controller**

```plain
/**
 * 基于线程池舱壁模式的Feign调用接口
 * 使用Resilience4j的THREADPOOL类型舱壁进行并发控制，当线程池满时执行降级方法
 * 
 * @param id 业务ID
 * @return CompletableFuture<String> 异步返回支付服务调用结果
 * 
 * @Bulkhead 注解说明：
 * - name: 舱壁实例名称，对应配置文件中的配置
 * - type: 舱壁类型，THREADPOOL表示使用线程池隔离
 * - fallbackMethod: 降级方法名，当线程池满或超时时执行
 * 
 * 工作机制：
 * 1. 方法执行进入独立线程池，不影响主线程池资源
 * 2. 当并发请求超过线程池容量时，触发降级逻辑
 * 3. 方法内部模拟3秒业务处理时间
 * 4. 通过CompletableFuture实现异步响应
 * 
 * 适用场景：保护下游支付服务，防止慢调用耗尽应用线程资源
 */
@GetMapping(value = "/feign/pay/bulkhead/{id}")
@Bulkhead(name = "cloud-payment-service", fallbackMethod = "myBulkheadPoolFallback", type = Bulkhead.Type.THREADPOOL)
public CompletableFuture<String> myBulkheadTHREADPOOL(@PathVariable("id") Integer id) {
    // 打印当前线程名称，标识方法开始执行
    System.out.println(Thread.currentThread().getName() + "\t" + "enter the method!!!");
    
    // 模拟业务处理耗时3秒，实际业务中可能是数据库操作、远程调用等
    try { 
        TimeUnit.SECONDS.sleep(3); 
    } catch (InterruptedException e) { 
        // 打印线程中断异常堆栈信息
        e.printStackTrace(); 
    }
    
    // 打印当前线程名称，标识方法执行结束
    System.out.println(Thread.currentThread().getName() + "\t" + "exist the method!!!");

    // 使用CompletableFuture异步调用Feign客户端，并添加舱壁类型标识
    return CompletableFuture.supplyAsync(() -> payFeignApi.myBulkhead(id) + "\t" + " Bulkhead.Type.THREADPOOL");
}

/**
 * 舱壁模式降级方法
 * 当线程池资源不足时返回友好提示
 * 
 * @param id 业务ID
 * @param t 异常信息（可选）
 * @return CompletableFuture<String> 异步返回降级结果
 */
public CompletableFuture<String> myBulkheadPoolFallback(Integer id, Throwable t) {
    // 异步返回系统繁忙提示信息，保持与主方法相同的返回类型
    return CompletableFuture.supplyAsync(() -> "Bulkhead.Type.THREADPOOL，系统繁忙，请稍后再试-----/(ㄒoㄒ)/~~");
}
```

Bulkhead.Type.THREADPOOL

**测试地址**

http://localhost/feign/pay/bulkhead/1 

http://localhost/feign/pay/bulkhead/2 

http://localhost/feign/pay/bulkhead/3 

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887299177-fce7bbaf-fec9-44c5-9d87-38131d130df3.png)

#### 7.5.3 限流(RateLimiter)
##### **7.5.3.1 官网**
[RateLimiter](https://resilience4j.readme.io/docs/ratelimiter)

中文

[https://github.com/lmhmhl/Resilience4j-Guides-Chinese/blob/main/core-modules/ratelimiter.md](https://github.com/lmhmhl/Resilience4j-Guides-Chinese/blob/main/core-modules/ratelimiter.md)

##### **7.5.3.2 是什么**
限流 就是限制最大访问流量。系统能提供的最大并发是有限的，同时来的请求又太多，就需要限流。 

比如商城秒杀业务，瞬时大量请求涌入，服务器忙不过就只好排队限流了，和去景点排队买票和去医院办理业务排队等号道理相同。

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887304800-ed01c5e4-fb45-4ff7-acdc-50ae08045202.png)

所谓限流，就是通过对并发访问/请求进行限速，或者对一个时间窗口内的请求进行限速，以保护应用系统，一旦达到限制速率则可以拒绝服务、排队或等待、降级等处理。

限流(频率控制)

##### **7.5.3.3 面试题：说说常见限流算法**
**1 漏斗算法(Leaky Bucket)**

**漏桶算法** 

一个固定容量的漏桶，按照设定常量固定速率流出水滴，类似医院打吊针，不管你源头流量多大，我设定匀速流出。 

如果流入水滴超出了桶的容量，则流入的水滴将会溢出了(被丢弃)，而漏桶容量是不变的。

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887303016-52e2c710-cf12-4117-90e2-130ac8af89ab.png)

缺点：

这里有两个变量，一个是桶的大小，支持流量突发增多时可以存多少的水（burst），另一个是水桶漏洞的大小（rate）。因为漏桶的漏出速率是固定的参数，所以，即使网络中不存在资源冲突（没有发生拥塞），漏桶算法也不能使流突发（burst）到端口速率。因此，漏桶算法对于存在突发特性的流量来说缺乏效率。

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887303449-d8ab0da0-5c28-47cd-805f-51521229310f.png)

**2 令牌桶算法(Token Bucket)**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887306196-da14f5fd-c7fd-4c6a-a235-3258606a7049.png)

SpringCloud默认使用该算法

**3 滚动时间窗(tumbling time window)**

允许固定数量的请求进入(比如1秒取4个数据相加，超过25值就over)超过数量就拒绝或者排队，等下一个时间段进入。

由于是在一个时间间隔内进行限制，如果用户在上个时间间隔结束前请求（但没有超过限制），同时在当前时间间隔刚开始请求（同样没超过限制），在各自的时间间隔内，这些请求都是正常的。下图统计了3次，but......

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887304048-65f5de9f-f850-4e23-ad23-aad7525b801e.png)

缺点：间隔临界的一段时间内的请求就会超过系统限制，可能导致系统被压垮

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887308015-c3ddd76a-7446-4f32-8214-2374e2dc5d8c.png)

假如设定1分钟最多可以请求100次某个接口，如12:00:00-12:00:59时间段内没有数据请求但12:00:59-12:01:00时间段内突然并发100次请求，紧接着瞬间跨入下一个计数周期计数器清零；在12:01:00-12:01:01内又有100次请求。那么也就是说在时间临界点左右可能同时有2倍的峰值进行请求，从而造成后台处理请求**加倍过载**的bug，导致系统运营能力不足，甚至导致系统崩溃，/(ㄒoㄒ)/~~

double kill

**4 滑动时间窗口(sliding time window)**

滑动时间窗口（sliding time window）

顾名思义，该时间窗口是滑动的。所以，从概念上讲，这里有两个方面的概念需要理解： 

- 窗口：需要定义窗口的大小

- 滑动：需要定义在窗口中滑动的大小，但理论上讲滑动的大小不能超过窗口大小

滑动窗口算法是把固定时间片进行划分并且随着时间移动，移动方式为开始时间点变为时间列表中的第2个时间点，结束时间点增加一个时间点，

不断重复，通过这种方式可以巧妙的避开计数器的临界点的问题。下图统计了5次

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887307997-53772afc-d572-4732-bfa9-4ebd6a683318.png)

##### **7.5.3.4 **cloud-provider-payment8001支付微服务修改PayCircuitController新增myRatelimit方法
```plain
//=========Resilience4j ratelimit 的例子
@GetMapping(value = "/pay/ratelimit/{id}")
public String myRatelimit(@PathVariable("id") Integer id)
{
    return "Hello, myRatelimit欢迎到来 inputId:  "+id+" \t " + IdUtil.simpleUUID();
}
```

##### **7.5.3.5 **PayFeignApi接口新增限流api方法
```plain
/**
 * Resilience4j Ratelimit 的例子
 * @param id
 * @return
 */
@GetMapping(value = "/pay/ratelimit/{id}")
public String myRatelimit(@PathVariable("id") Integer id);
```

##### **7.5.3.6 **修改cloud-consumer-feign-order80
**POM**

```plain
<!--resilience4j-ratelimiter-->
<dependency>
    <groupId>io.github.resilience4j</groupId>
    <artifactId>resilience4j-ratelimiter</artifactId>
</dependency>
```

**YML**

```plain
####resilience4j ratelimiter 限流的例子
resilience4j:
  ratelimiter:
    configs:
      default:
        limitForPeriod: 2 #在一次刷新周期内，允许执行的最大请求数
        limitRefreshPeriod: 1s # 限流器每隔limitRefreshPeriod刷新一次，将允许处理的最大请求数量重置为limitForPeriod
        timeout-duration: 1 # 线程等待权限的默认等待时间
    instances:
        cloud-payment-service:
          baseConfig: default
```

**order的controller**

```plain
@GetMapping(value = "/feign/pay/ratelimit/{id}")
@RateLimiter(name = "cloud-payment-service",fallbackMethod = "myRatelimitFallback")
public String myBulkhead(@PathVariable("id") Integer id)
{
    return payFeignApi.myRatelimit(id);
}
public String myRatelimitFallback(Integer id,Throwable t)
{
    return "你被限流了，禁止访问/(ㄒoㄒ)/~~";
}
```

@RateLimiter 

##### **7.5.3.7 **测试
[http://localhost/feign/pay/ratelimit/11](http://localhost/feign/pay/ratelimit/11)

结果

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887307479-5c30d0bd-a28b-4f7f-8d75-398f63de0b4f.png)

刷新上述地址，正常后F5按钮狂刷一会儿，停止刷新看到被限流的效果

## 8 Sleuth(Micrometer)+ZipKin分布式链路追踪
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887346327-3176d985-6edf-4876-9963-ec6a8679f5cc.png)

### 8.1 Sleuth目前也进入维护模式
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887346302-44b8d721-6d62-49d8-bd29-b6f8afd8a13e.png)

### 8.2 分布式链路追踪概述
#### 8.2.1 为什么会出现这个技术？需要解决哪些问题？
在微服务框架中，一个由客户端发起的请求在后端系统中会经过多个不同的的服务节点调用来协同产生最后的请求结果，每一个前段请求都会形成一条复杂的分布式服务调用链路，链路中的任何一环出现高延时或错误都会引起整个请求最后的失败。

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887346013-e1b5cff0-5ff3-431c-a9fc-e58b74d2940a.png)

#### 8.2.2 随着问题的复杂化+微服务的增多+调用链条的变长。画面不要太美丽....../(ㄒoㄒ)/~~
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887347988-4b81746b-751a-42ab-8c58-48a478952a3e.png)

#### 8.2.3 在分布式与微服务场景下需要解决的问题
在分布式与微服务场景下，我们需要解决如下问题：

在大规模分布式与微服务集群下，如何实时观测系统的整体调用链路情况。

在大规模分布式与微服务集群下，如何快速发现并定位到问题。

在大规模分布式与微服务集群下，如何尽可能精确的判断故障对系统的影响范围与影响程度。

在大规模分布式与微服务集群下，如何尽可能精确的梳理出服务之间的依赖关系，并判断出服务之间的依赖关系是否合理。

在大规模分布式与微服务集群下，如何尽可能精确的分析整个系统调用链路的性能与瓶颈点。

在大规模分布式与微服务集群下，如何尽可能精确的分析系统的存储瓶颈与容量规划。

上述问题就是我们的落地议题答案：

分布式链路追踪技术要解决的问题，分布式链路追踪（Distributed Tracing），就是将一次分布式请求还原成调用链路，进行日志记录，性能监控并将一次分布式请求的调用情况集中展示。比如各个服务节点上的耗时、请求具体到达哪台机器上、每个服务节点的请求状态等等。

### 8.3 新一代Spring Cloud Sleuth：Micrometer
#### **8.3.1 (官网重要提示)**
**新一代Sleuth**

sleuth被micrometer替代

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887346079-a8e843dd-0408-4905-b9c3-e5cd1c13c5fd.png)

**官网**

[Spring Cloud Sleuth](https://spring.io/projects/spring-cloud-sleuth#overview)

github：

[https://github.com/spring-cloud/spring-cloud-sleuth](https://github.com/spring-cloud/spring-cloud-sleuth)

**说明**

**老项目还能用Sleuth开发吗**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887350051-913de0aa-13fa-462d-9e8c-2c1593db0509.png)

**版本注意**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887351129-36446508-2ca4-439b-b998-20e1747b21c1.png)

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887351172-5f120d74-d71c-4062-8df2-ef56cd9c03fa.png)

#### 8.3.2 zipkin那？
Spring Cloud Sleuth(micrometer)提供了一套完整的分布式链路追踪（Distributed Tracing）解决方案且兼容支持了zipkin展现

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887351182-122669d6-ee85-4d30-934d-c1f2b449a24d.png)

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887352141-d78c1105-733f-4de1-a579-42697bab3367.png)

#### 8.3.3 小总结
将一次分布式请求还原成调用链路，进行日志记录和性能监控，并将一次分布式请求的调用情况集中web展示

#### 8.3.4 行业内比较成熟的其它分布式链路追踪技术解决方案
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887355281-f7cca0e7-184f-4bac-8353-fad0b68498f1.png)

### 8.4 分布式链路追踪原理
**假定三个微服务调用的链路如下图所示：Service 1 调用 Service 2，Service 2 调用 Service 3 和 Service 4。**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887356763-7be491e3-45bb-474c-97c9-81a59c1b2e8c.png)

**上一步完整的调用链路**

那么一条链路追踪会在每个服务调用的时候加上Trace ID 和 Span ID

链路通过TraceId唯一标识，

Span标识发起的请求信息，各span通过parent id 关联起来 (Span:表示调用链路来源，通俗的理解span就是一次请求信息)

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887357947-2cc5ef07-ec26-4bb6-8d9a-e2d32d3662ab.png)

**彻底把链路追踪整明白**

一条链路通过Trace Id唯一标识，Span标识发起的请求信息，各span通过parent id 关联起来

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887357941-63b3476e-4dd7-4eff-be1b-f0bdacc29f99.png)

| 1 | 第一个节点：Span ID = A，Parent ID = null，Service 1 接收到请求。 |
| :--- | :--- |
| 2 | 第二个节点：Span ID = B，Parent ID= A，Service 1 发送请求到 Service 2 返回响应给Service 1 的过程。 |
| 3 | 第三个节点：Span ID = C，Parent ID= B，Service 2 的 中间解决过程。 |
| 4 | 第四个节点：Span ID = D，Parent ID= C，Service 2 发送请求到 Service 3 返回响应给Service 2 的过程。 |
| 5 | 第五个节点：Span ID = E，Parent ID= D，Service 3 的中间解决过程。 |
| 6 | 第六个节点：Span ID = F，Parent ID= C，Service 3 发送请求到 Service 4 返回响应给 Service 3 的过程。 |
| 7 | 第七个节点：Span ID = G，Parent ID= F，Service 4 的中间解决过程。 |
| 8 | 通过 Parent ID 就可找到父节点，整个链路即可以进行跟踪追溯了。 |


### 8.5 Zipkin
#### **8.5.1 官网**
[OpenZipkin · A distributed tracing system](https://zipkin.io/)

#### **8.5.2 是什么**
**ZipKin概述**

Zipkin是一种分布式链路跟踪系统图形化的工具，Zipkin 是 Twitter 开源的分布式跟踪系统，能够收集微服务运行过程中的实时调用链路信息，并能够将这些调用链路信息展示到Web图形化界面上供开发人员分析，开发人员能够从ZipKin中分析出调用链路中的性能瓶颈，识别出存在问题的应用程序，进而定位问题和解决问题。

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887357961-904775ec-6aa7-4630-9043-3dfffbb9d566.png)

#### **8.5.3 **Zipkin为什么出现？
**单有Sleuth(Micrometer)行不行？**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887361114-9210317a-e507-40a7-8e55-c67002701419.png)

说明：

当没有配置 Sleuth 链路追踪的时候，INFO 信息里面是 [passjava-question,,,]，后面跟着三个空字符串。

当配置了 Sleuth 链路追踪的时候，追踪到的信息是 [passjava-question,504a5360ca906016,e55ff064b3941956,false] ，第一个是 Trace ID，第二个是 Span ID。**只有日志没有图，观看不方便，不美观，so，**引入图形化Zipkin链路监控让你好看，O(∩_∩)O

#### **8.5.4 **下载+安装+运行一套带走
**下载主页**

[Quickstart · OpenZipkin](https://zipkin.io/pages/quickstart)

支持3个方式

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887363075-ae6482c2-52d9-4703-a725-3fa120fe1e53.png)

**下载地址**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887364361-20f86382-2030-4d3e-9e96-f60dbb3af259.png)

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887363831-2c94d379-7dee-41f3-a3e9-d07b4e60e3ae.png)

2023.12，版本名称

zipkin-server-3.0.0-rc0-exec.jar

**运行jar**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887363850-2b19f173-0557-4e6e-9b45-bd7cd5ef6ab7.png)

**运行控制台**

[http://localhost:9411/zipkin/](http://localhost:9411/zipkin/)

### 8.6 Micrometer+ZipKin搭建链路监控案例步骤
#### 8.6.1 Micrometer+ZipKin两者各自的分工
**Micrometer**

数据采样

**ZipKin**

图形展示

#### **8.6.2 步骤**
**总体父工程POM**

**本案例**

```plain
<!--micrometer-tracing-bom导入链路追踪版本中心  1-->
<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-tracing-bom</artifactId>
    <version>${micrometer-tracing.version}</version>
    <type>pom</type>
    <scope>import</scope>
</dependency>
<!--micrometer-tracing指标追踪  2-->
<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-tracing</artifactId>
    <version>${micrometer-tracing.version}</version>
</dependency>
<!--micrometer-tracing-bridge-brave适配zipkin的桥接包 3-->
<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-tracing-bridge-brave</artifactId>
    <version>${micrometer-tracing.version}</version>
</dependency>
<!--micrometer-observation 4-->
<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-observation</artifactId>
    <version>${micrometer-observation.version}</version>
</dependency>
<!--feign-micrometer 5-->
<dependency>
    <groupId>io.github.openfeign</groupId>
    <artifactId>feign-micrometer</artifactId>
    <version>${feign-micrometer.version}</version>
</dependency>
<!--zipkin-reporter-brave 6-->
<dependency>
    <groupId>io.zipkin.reporter2</groupId>
    <artifactId>zipkin-reporter-brave</artifactId>
    <version>${zipkin-reporter-brave.version}</version>
</dependency>
```

引入的jar包分别是什么意思

由于Micrometer Tracing是一个门面工具自身并没有实现完整的链路追踪系统，具体的链路追踪另外需要引入的是第三方链路追踪系统的依赖：

| 1 | micrometer-tracing-bom | 导入链路追踪版本中心，体系化说明 |
| :--- | :--- | :--- |
| 2 | micrometer-tracing | 指标追踪 |
| 3 | micrometer-tracing-bridge-brave | 一个Micrometer模块，用于与分布式跟踪工具 Brave 集成，以收集应用程序的分布式跟踪数据。Brave是一个开源的分布式跟踪工具，它可以帮助用户在分布式系统中跟踪请求的流转，它使用一种称为"跟踪上下文"的机制，将请求的跟踪信息存储在请求的头部，然后将请求传递给下一个服务。在整个请求链中，Brave会将每个服务处理请求的时间和其他信息存储到跟踪数据中，以便用户可以了解整个请求的路径和性能。 |
| 4 | micrometer-observation | 一个基于度量库 Micrometer的观测模块，用于收集应用程序的度量数据。 |
| 5 | feign-micrometer | 一个Feign HTTP客户端的Micrometer模块，用于收集客户端请求的度量数据。 |
| 6 | zipkin-reporter-brave | 一个用于将 Brave 跟踪数据报告到Zipkin 跟踪系统的库。 |


补充包：spring-boot-starter-actuator   SpringBoot框架的一个模块用于监视和管理应用程序

**all**

```plain
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.atguigu.cloud</groupId>
    <artifactId>mscloudV6</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>pom</packaging>
    <modules>
        <module>cloud-provider-payment8001</module>
        <module>cloud-consumer-order80</module>
        <module>cloud-api-commons</module>
        <module>cloud-provider-payment8002</module>
        <module>cloud-consumer-feign-order80</module>
    </modules>

    <properties>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <hutool.version>5.8.22</hutool.version>
        <lombok.version>1.18.26</lombok.version>
        <druid.version>1.1.20</druid.version>
        <mybatis.springboot.version>3.0.2</mybatis.springboot.version>
        <mysql.version>8.0.11</mysql.version>
        <swagger3.version>2.2.0</swagger3.version>
        <mapper.version>4.2.3</mapper.version>
        <fastjson2.version>2.0.41</fastjson2.version>
        <persistence-api.version>1.0.2</persistence-api.version>
        <spring.boot.test.version>3.1.5</spring.boot.test.version>
        <spring.boot.version>3.2.0</spring.boot.version>
        <spring.cloud.version>2023.0.0</spring.cloud.version>
        <spring.cloud.alibaba.version>2022.0.0.0-RC2</spring.cloud.alibaba.version>
        <micrometer-tracing.version>1.2.0</micrometer-tracing.version>
        <micrometer-observation.version>1.12.0</micrometer-observation.version>
        <feign-micrometer.version>12.5</feign-micrometer.version>
        <zipkin-reporter-brave.version>2.17.0</zipkin-reporter-brave.version>
    </properties>

    <dependencyManagement>
        <dependencies>
            <!--springboot 3.2.0-->
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-parent</artifactId>
                <version>${spring.boot.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <!--springcloud 2023.0.0-->
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring.cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <!--springcloud alibaba 2022.0.0.0-RC2-->
            <dependency>
                <groupId>com.alibaba.cloud</groupId>
                <artifactId>spring-cloud-alibaba-dependencies</artifactId>
                <version>${spring.cloud.alibaba.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <!--SpringBoot集成mybatis-->
            <dependency>
                <groupId>org.mybatis.spring.boot</groupId>
                <artifactId>mybatis-spring-boot-starter</artifactId>
                <version>${mybatis.springboot.version}</version>
            </dependency>
            <!--Mysql数据库驱动8 -->
            <dependency>
                <groupId>mysql</groupId>
                <artifactId>mysql-connector-java</artifactId>
                <version>${mysql.version}</version>
            </dependency>
            <!--SpringBoot集成druid连接池-->
            <dependency>
                <groupId>com.alibaba</groupId>
                <artifactId>druid-spring-boot-starter</artifactId>
                <version>${druid.version}</version>
            </dependency>
            <!--通用Mapper4之tk.mybatis-->
            <dependency>
                <groupId>tk.mybatis</groupId>
                <artifactId>mapper</artifactId>
                <version>${mapper.version}</version>
            </dependency>
            <!--persistence-->
            <dependency>
                <groupId>javax.persistence</groupId>
                <artifactId>persistence-api</artifactId>
                <version>${persistence-api.version}</version>
            </dependency>
            <!-- fastjson2 -->
            <dependency>
                <groupId>com.alibaba.fastjson2</groupId>
                <artifactId>fastjson2</artifactId>
                <version>2.0.40</version>
            </dependency>
            <!-- swagger3 调用方式 http://你的主机IP地址:5555/swagger-ui/index.html -->
            <dependency>
                <groupId>org.springdoc</groupId>
                <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
                <version>${swagger3.version}</version>
            </dependency>
            <!--hutool-->
            <dependency>
                <groupId>cn.hutool</groupId>
                <artifactId>hutool-all</artifactId>
                <version>${hutool.version}</version>
            </dependency>
            <!--lombok-->
            <dependency>
                <groupId>org.projectlombok</groupId>
                <artifactId>lombok</artifactId>
                <version>${lombok.version}</version>
                <optional>true</optional>
            </dependency>
            <!-- spring-boot-starter-test -->
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-test</artifactId>
                <version>${spring.boot.test.version}</version>
                <scope>test</scope>
            </dependency>
            <!--micrometer-tracing-bom导入链路追踪版本中心  1-->
            <dependency>
                <groupId>io.micrometer</groupId>
                <artifactId>micrometer-tracing-bom</artifactId>
                <version>${micrometer-tracing.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <!--micrometer-tracing指标追踪  2-->
            <dependency>
                <groupId>io.micrometer</groupId>
                <artifactId>micrometer-tracing</artifactId>
                <version>${micrometer-tracing.version}</version>
            </dependency>
            <!--micrometer-tracing-bridge-brave适配zipkin的桥接包 3-->
            <dependency>
                <groupId>io.micrometer</groupId>
                <artifactId>micrometer-tracing-bridge-brave</artifactId>
                <version>${micrometer-tracing.version}</version>
            </dependency>
            <!--micrometer-observation 4-->
            <dependency>
                <groupId>io.micrometer</groupId>
                <artifactId>micrometer-observation</artifactId>
                <version>${micrometer-observation.version}</version>
            </dependency>
            <!--feign-micrometer 5-->
            <dependency>
                <groupId>io.github.openfeign</groupId>
                <artifactId>feign-micrometer</artifactId>
                <version>${feign-micrometer.version}</version>
            </dependency>
            <!--zipkin-reporter-brave 6-->
            <dependency>
                <groupId>io.zipkin.reporter2</groupId>
                <artifactId>zipkin-reporter-brave</artifactId>
                <version>${zipkin-reporter-brave.version}</version>
            </dependency>

        </dependencies>
    </dependencyManagement>
</project>
```

**服务提供者8001**

**cloud-provider-payment8001**

**POM**

```plain
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>com.atguigu.cloud</groupId>
        <artifactId>mscloudV6</artifactId>
        <version>1.0-SNAPSHOT</version>
    </parent>

    <artifactId>cloud-provider-payment8001</artifactId>

    <properties>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>



    <dependencies>
        <!--micrometer-tracing指标追踪  1-->
        <dependency>
            <groupId>io.micrometer</groupId>
            <artifactId>micrometer-tracing</artifactId>
        </dependency>
        <!--micrometer-tracing-bridge-brave适配zipkin的桥接包 2-->
        <dependency>
            <groupId>io.micrometer</groupId>
            <artifactId>micrometer-tracing-bridge-brave</artifactId>
        </dependency>
        <!--micrometer-observation 3-->
        <dependency>
            <groupId>io.micrometer</groupId>
            <artifactId>micrometer-observation</artifactId>
        </dependency>
        <!--feign-micrometer 4-->
        <dependency>
            <groupId>io.github.openfeign</groupId>
            <artifactId>feign-micrometer</artifactId>
        </dependency>
        <!--zipkin-reporter-brave 5-->
        <dependency>
            <groupId>io.zipkin.reporter2</groupId>
            <artifactId>zipkin-reporter-brave</artifactId>
        </dependency>
        <!--SpringCloud consul config-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-consul-config</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-bootstrap</artifactId>
        </dependency>
        <!--SpringCloud consul discovery -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-consul-discovery</artifactId>
        </dependency>
        <!-- 引入自己定义的api通用包 -->
        <dependency>
            <groupId>com.atguigu.cloud</groupId>
            <artifactId>cloud-api-commons</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
        <!--SpringBoot通用依赖模块-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <!--SpringBoot集成druid连接池-->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid-spring-boot-starter</artifactId>
        </dependency>
        <!-- Swagger3 调用方式 http://你的主机IP地址:5555/swagger-ui/index.html -->
        <dependency>
            <groupId>org.springdoc</groupId>
            <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
        </dependency>
        <!--mybatis和springboot整合-->
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
        </dependency>
        <!--Mysql数据库驱动8 -->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
        </dependency>
        <!--persistence-->
        <dependency>
            <groupId>javax.persistence</groupId>
            <artifactId>persistence-api</artifactId>
        </dependency>
        <!--通用Mapper4-->
        <dependency>
            <groupId>tk.mybatis</groupId>
            <artifactId>mapper</artifactId>
        </dependency>
        <!--hutool-->
        <dependency>
            <groupId>cn.hutool</groupId>
            <artifactId>hutool-all</artifactId>
        </dependency>
        <!-- fastjson2 -->
        <dependency>
            <groupId>com.alibaba.fastjson2</groupId>
            <artifactId>fastjson2</artifactId>
        </dependency>
        <!--lombok-->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.18.28</version>
            <scope>provided</scope>
        </dependency>
        <!--test-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
```

**YML**

```plain
server:
  port: 8001

# ==========applicationName + druid-mysql8 driver===================
spring:
  datasource:
    type: com.alibaba.druid.pool.DruidDataSource
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/db2024?characterEncoding=utf8&useSSL=false&serverTimezone=GMT%2B8&rewriteBatchedStatements=true&allowPublicKeyRetrieval=true
    username: root
    password: 123456
  profiles:
    active: dev # 多环境配置加载内容dev/prod,不写就是默认default配置

# ========================mybatis===================
mybatis:
  mapper-locations: classpath:mapper/*.xml
  type-aliases-package: com.atguigu.cloud.entities
  configuration:
    map-underscore-to-camel-case: true


# ========================zipkin===================
management:
  zipkin:
    tracing:
      endpoint: http://localhost:9411/api/v2/spans
  tracing:
    sampling:
      probability: 1.0 #采样率默认为0.1(0.1就是10次只能有一次被记录下来)，值越大收集越及时。
```

**新建业务类PayMicrometerController**

```plain
package com.atguigu.cloud.controller;

import cn.hutool.core.util.IdUtil;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class PayMicrometerController
{
    /**
     * Micrometer(Sleuth)进行链路监控的例子
     * @param id
     * @return
     */
    @GetMapping(value = "/pay/micrometer/{id}")
    public String myMicrometer(@PathVariable("id") Integer id)
    {
        return "Hello, 欢迎到来myMicrometer inputId:  "+id+" \t    服务返回:" + IdUtil.simpleUUID();
    }
}
```

**Api接口PayFeignApi**

```plain
package com.atguigu.cloud.apis;

import com.atguigu.cloud.entities.PayDTO;
import com.atguigu.cloud.resp.ResultData;
import io.github.resilience4j.bulkhead.annotation.Bulkhead;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;

@FeignClient(value = "cloud-payment-service")
public interface PayFeignApi
{
    /**
     * 新增一条支付相关流水记录
     * @param payDTO
     * @return
     */
    @PostMapping("/pay/add")
    public ResultData addPay(@RequestBody PayDTO payDTO);

    /**
     * 按照主键记录查询支付流水信息
     * @param id
     * @return
     */
    @GetMapping("/pay/get/{id}")
    public ResultData getPayInfo(@PathVariable("id") Integer id);

    /**
     * openfeign天然支持负载均衡演示
     * @return
     */
    @GetMapping(value = "/pay/get/info")
    public String mylb();

    /**
     * Resilience4j CircuitBreaker 的例子
     * @param id
     * @return
     */
    @GetMapping(value = "/pay/circuit/{id}")
    public String myCircuit(@PathVariable("id") Integer id);

    /**
     * Resilience4j Bulkhead 的例子
     * @param id
     * @return
     */
    @GetMapping(value = "/pay/bulkhead/{id}")
    public String myBulkhead(@PathVariable("id") Integer id);

    /**
     * Resilience4j Ratelimit 的例子
     * @param id
     * @return
     */
    @GetMapping(value = "/pay/ratelimit/{id}")
    public String myRatelimit(@PathVariable("id") Integer id);


    /**
     * Micrometer(Sleuth)进行链路监控的例子
     * @param id
     * @return
     */
    @GetMapping(value = "/pay/micrometer/{id}")
    public String myMicrometer(@PathVariable("id") Integer id);
}
```

**服务调用者80**

**cloud-consumer-feign-order80**

**POM**

```plain
<!--micrometer-tracing指标追踪  1-->
    <dependency>
        <groupId>io.micrometer</groupId>
        <artifactId>micrometer-tracing</artifactId>
    </dependency>
    <!--micrometer-tracing-bridge-brave适配zipkin的桥接包 2-->
    <dependency>
        <groupId>io.micrometer</groupId>
        <artifactId>micrometer-tracing-bridge-brave</artifactId>
    </dependency>
    <!--micrometer-observation 3-->
    <dependency>
        <groupId>io.micrometer</groupId>
        <artifactId>micrometer-observation</artifactId>
    </dependency>
    <!--feign-micrometer 4-->
    <dependency>
        <groupId>io.github.openfeign</groupId>
        <artifactId>feign-micrometer</artifactId>
    </dependency>
    <!--zipkin-reporter-brave 5-->
    <dependency>
        <groupId>io.zipkin.reporter2</groupId>
        <artifactId>zipkin-reporter-brave</artifactId>
    </dependency>
```

All 

```plain
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>com.atguigu.cloud</groupId>
        <artifactId>mscloudV6</artifactId>
        <version>1.0-SNAPSHOT</version>
    </parent>

    <artifactId>cloud-consumer-feign-order80</artifactId>

    <properties>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>


    <dependencies>
        <!--micrometer-tracing指标追踪  1-->
        <dependency>
            <groupId>io.micrometer</groupId>
            <artifactId>micrometer-tracing</artifactId>
        </dependency>
        <!--micrometer-tracing-bridge-brave适配zipkin的桥接包 2-->
        <dependency>
            <groupId>io.micrometer</groupId>
            <artifactId>micrometer-tracing-bridge-brave</artifactId>
        </dependency>
        <!--micrometer-observation 3-->
        <dependency>
            <groupId>io.micrometer</groupId>
            <artifactId>micrometer-observation</artifactId>
        </dependency>
        <!--feign-micrometer 4-->
        <dependency>
            <groupId>io.github.openfeign</groupId>
            <artifactId>feign-micrometer</artifactId>
        </dependency>
        <!--zipkin-reporter-brave 5-->
        <dependency>
            <groupId>io.zipkin.reporter2</groupId>
            <artifactId>zipkin-reporter-brave</artifactId>
        </dependency>
        <!--resilience4j-ratelimiter-->
        <dependency>
            <groupId>io.github.resilience4j</groupId>
            <artifactId>resilience4j-ratelimiter</artifactId>
        </dependency>
        <!--resilience4j-bulkhead-->
        <dependency>
            <groupId>io.github.resilience4j</groupId>
            <artifactId>resilience4j-bulkhead</artifactId>
        </dependency>
        <!--resilience4j-circuitbreaker-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-circuitbreaker-resilience4j</artifactId>
        </dependency>
        <!-- 由于断路保护等需要AOP实现，所以必须导入AOP包 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-aop</artifactId>
        </dependency>
        <!-- httpclient5-->
        <dependency>
            <groupId>org.apache.httpcomponents.client5</groupId>
            <artifactId>httpclient5</artifactId>
            <version>5.3</version>
        </dependency>
        <!-- feign-hc5-->
        <dependency>
            <groupId>io.github.openfeign</groupId>
            <artifactId>feign-hc5</artifactId>
            <version>13.1</version>
        </dependency>
        <!--openfeign-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>
        <!--SpringCloud consul discovery-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-consul-discovery</artifactId>
        </dependency>
        <!-- 引入自己定义的api通用包 -->
        <dependency>
            <groupId>com.atguigu.cloud</groupId>
            <artifactId>cloud-api-commons</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
        <!--web + actuator-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <!--lombok-->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <!--hutool-all-->
        <dependency>
            <groupId>cn.hutool</groupId>
            <artifactId>hutool-all</artifactId>
        </dependency>
        <!--fastjson2-->
        <dependency>
            <groupId>com.alibaba.fastjson2</groupId>
            <artifactId>fastjson2</artifactId>
        </dependency>
        <!-- swagger3 调用方式 http://你的主机IP地址:5555/swagger-ui/index.html -->
        <dependency>
            <groupId>org.springdoc</groupId>
            <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```

**YML**

```plain
# zipkin图形展现地址和采样率设置
management:
  zipkin:
    tracing:
      endpoint: http://localhost:9411/api/v2/spans
  tracing:
    sampling:
      probability: 1.0 #采样率默认为0.1(0.1就是10次只能有一次被记录下来)，值越大收集越及时。
```

**新建业务类OrderMicrometerController**

```plain
package com.atguigu.cloud.controller;

import com.atguigu.cloud.apis.PayFeignApi;
import jakarta.annotation.Resource;
import lombok.extern.slf4j.Slf4j;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;

/**
 * Micrometer 替代 Sleuth
 */
@RestController
@Slf4j
public class OrderMicrometerController
{
    @Resource
    private PayFeignApi payFeignApi;

    @GetMapping(value = "/feign/micrometer/{id}")
    public String myMicrometer(@PathVariable("id") Integer id)
    {
        return payFeignApi.myMicrometer(id);
    }
}
```

#### **8.6.3 测试**
**本次案例，默认已经成功启动Zipkin**

**依次启动8001/80两个微服务并注册进入Consul**

**测试地址**

[http://localhost/feign/micrometer/1](http://localhost/feign/micrometer/1)

**打开浏览器访问：http://localhost:9411**

**会出现以下界面**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887366099-d3f96d79-95df-4a88-853e-e323b5fa9f1e.png)

点击【SHOW】按钮查看

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887367265-047f17f7-3ff6-4f88-8e3c-dbd863047603.png)

**查看依赖关系**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887368897-ef0c59bf-dd5c-4f55-9160-c76569701bd3.png)

## 9 Gateway新一代网关 
### 9.1 概述


#### **9.1.1 是什么**
**官网**

Gateway是在Spring生态系统之上构建的API网关服务，基于Spring6，Spring Boot 3和Project Reactor等技术。它旨在为微服务架构提供一种简单有效的统一的 API 路由管理方式，并为它们提供跨领域的关注点，例如：安全性、监控/度量和恢复能力。

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887427487-6b46d57f-e3e9-4272-8d09-f59aa70d7e58.png)

[Spring Cloud Gateway](https://docs.spring.io/spring-cloud-gateway/docs/4.0.4/reference/html/)

**体系定位**

Cloud全家桶中有个很重要的组件就是网关，在1.x版本中都是采用的Zuul网关；

但在2.x版本中，zuul的升级一直跳票，SpringCloud最后自己研发了一个网关SpringCloud Gateway替代Zuul，

那就是SpringCloud Gateway一句话：gateway是原zuul1.x版的替代

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887427603-88cc4610-65b5-45ca-a240-cee40f28db2f.png)

#### **9.1.2 微服务架构中网关在哪里**
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887427937-f9c431a1-3122-415d-b6dc-7ccbc729110c.png)

#### **9.1.3 能干嘛**
反向代理

鉴权

流量控制

熔断

日志监控

#### **9.1.4 总结**
| Spring Cloud Gateway组件的核心是一系列的过滤器，通过这些过滤器可以将客户端发送的请求转发(路由)到对应的微服务。 Spring Cloud Gateway是加在整个微服务最前沿的防火墙和代理器，隐藏微服务结点IP端口信息，从而加强安全保护。Spring Cloud Gateway本身也是一个微服务，需要注册进服务注册中心。 |
| :--- |


![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887427715-301c70ec-2dcb-4eac-9213-fd21e28a59cb.png)

### 9.2 Gateway三大核心
**1 总述官网**

![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887432490-ac804800-b55f-48a1-a089-c09b05a74592.png)

**2 分**

**Route(路由)**

路由是构建网关的基本模块，它由ID，目标URI，一系列的断言和过滤器组成，如果断言为true则匹配该路由

**Predicate(断言)**

参考的是Java8的java.util.function.Predicate

开发人员可以匹配HTTP请求中的所有内容(例如请求头或请求参数)，如果请求与断言相匹配则进行路由

**Filter(过滤)**

指的是Spring框架中GatewayFilter的实例，使用过滤器，可以在请求被路由前或者之后对请求进行修改。

**3 总结**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887435676-de13bcf9-d1bf-4f89-91a7-c804dee4c79f.png)

web前端请求，通过一些匹配条件，定位到真正的服务节点。并在这个转发过程的前后，进行一些精细化控制。

predicate就是我们的匹配条件；

filter，就可以理解为一个无所不能的拦截器。有了这两个元素，再加上目标uri，就可以实现一个具体的路由了

### 9.3 Gateway工作流程
**官网总结**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887435431-a5581c9a-c971-4bf4-a482-52fd911c5aab.png)

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887435796-264b10ef-3214-43b7-b1d2-b5c5600a53ba.png)

客户端向 Spring Cloud Gateway 发出请求。然后在 Gateway Handler Mapping 中找到与请求相匹配的路由，将其发送到 Gateway Web Handler。Handler 再通过指定的过滤器链来将请求发送到我们实际的服务执行业务逻辑，然后返回。

过滤器之间用虚线分开是因为过滤器可能会在发送代理请求之前(Pre)或之后(Post)执行业务逻辑。

在“pre”类型的过滤器可以做参数校验、权限校验、流量监控、日志输出、协议转换等;

在“post”类型的过滤器中可以做响应内容、响应头的修改，日志的输出，流量监控等有着非常重要的作用。

**核心逻辑**

路由转发+断言判断+执行过滤器链

### 9.4 入门配置
**建Module**

cloud-gateway9527

**改POM**

```plain
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <parent>
    <groupId>com.atguigu.cloud</groupId>
    <artifactId>mscloudV5</artifactId>
    <version>1.0-SNAPSHOT</version>
  </parent>

  <artifactId>cloud-gateway9527</artifactId>

  <properties>
    <maven.compiler.source>17</maven.compiler.source>
    <maven.compiler.target>17</maven.compiler.target>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
  </properties>


  <dependencies>
    <!--gateway-->
    <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-gateway</artifactId>
    </dependency>
    <!--服务注册发现consul discovery,网关也要注册进服务注册中心统一管控-->
    <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-consul-discovery</artifactId>
    </dependency>
    <!-- 指标监控健康检查的actuator,网关是响应式编程删除掉spring-boot-starter-web dependency-->
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
  </dependencies>

  <build>
    <plugins>
      <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
      </plugin>
    </plugins>
  </build>
</project>
```

**写YML**

```plain
server:
  port: 9527

spring:
  application:
    name: cloud-gateway #以微服务注册进consul或nacos服务列表内
  cloud:
    consul: #配置consul地址
      host: localhost
      port: 8500
      discovery:
        prefer-ip-address: true
        service-name: ${spring.application.name}
```

**主启动**

```plain
package com.atguigu.cloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;

/**
 * @auther zzyy
 * @create 2023-11-20 12:38
 */
@SpringBootApplication
@EnableDiscoveryClient //服务注册和发现
public class Main9527
{
    public static void main(String[] args)
    {
        SpringApplication.run(Main9527.class,args);
    }
}
```

**业务类**

无，不写任何业务代码，网关和业务无关

**测试**

先启动8500服务中心Consul

再启动9527网关入驻

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887435490-a22e882b-ef0f-459d-b1c4-9b8844280caa.png)

### 9.5 9527网关如何做路由映射
#### 9.5.1 9527网关如何做路由映射那？？？
**诉求**

我们目前不想暴露8001端口，希望在8001真正的支付微服务外面套一层9527网关

**8001新建PayGateWayController**

```plain
package com.atguigu.cloud.controller;

import cn.hutool.core.util.IdUtil;
import com.atguigu.cloud.entities.Pay;
import com.atguigu.cloud.resp.ResultData;
import com.atguigu.cloud.service.PayService;
import io.swagger.v3.oas.annotations.Operation;
import jakarta.annotation.Resource;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;

import java.util.concurrent.TimeUnit;

@RestController
public class PayGateWayController
{
    @Resource
    PayService payService;

    @GetMapping(value = "/pay/gateway/get/{id}")
    public ResultData<Pay> getById(@PathVariable("id") Integer id)
    {
        Pay pay = payService.getById(id);
        return ResultData.success(pay);
    }

    @GetMapping(value = "/pay/gateway/info")
    public ResultData<String> getGatewayInfo()
    {
        return ResultData.success("gateway info test："+ IdUtil.simpleUUID());
    }
}
```

**启动8001支付**

**8001自测通过**

[http://localhost:8001/pay/gateway/get/1](http://localhost:8001/pay/gateway/get/1)

[http://localhost:8001/pay/gateway/info](http://localhost:8001/pay/gateway/info)

#### **9.5.2 9527网关YML新增配置**
```plain
server:
port: 9527

spring:
application:
name: cloud-gateway #以微服务注册进consul或nacos服务列表内
  cloud:
  consul: #配置consul地址
    host: localhost
	port: 8500
	discovery:
	prefer-ip-address: true
	service-name: ${spring.application.name}
	gateway:
	routes:
	- id: pay_routh1 #pay_routh1                #路由的ID(类似mysql主键ID)，没有固定规则但要求唯一，建议配合服务名
	uri: http://localhost:8001                #匹配后提供服务的路由地址
	predicates:
	- Path=/pay/gateway/get/**              # 断言，路径相匹配的进行路由
	
	
	- id: pay_routh2 #pay_routh2                #路由的ID(类似mysql主键ID)，没有固定规则但要求唯一，建议配合服务名
	uri: http://localhost:8001                #匹配后提供服务的路由地址
	predicates:
- Path=/pay/gateway/info/**              # 断言，路径相匹配的进行路由
```

#### 9.5.3 **测试1**
**启动Consul8500服务**

**启动8001支付**

**启动9527网关**

**访问说明**

**添加网关前**

[http://localhost:8001/pay/gateway/get/1](http://localhost:8001/pay/gateway/get/1)

[http://localhost:8001/pay/gateway/info](http://localhost:8001/pay/gateway/info)

**隐真示假，映射说明**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887435681-dcd7219d-b972-4bf2-a4da-a5329c972582.png)

**添加网关后**

[http://localhost:9527/pay/gateway/get/1](http://localhost:9527/pay/gateway/get/1)

[http://localhost:9527/pay/gateway/info](http://localhost:9527/pay/gateway/info)

**目前8001支付微服务前面添加GateWay成功**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887441922-65556986-65f1-48bc-8150-180c3f92cda6.png)

GateWay9527 → Pay8001

#### 9.5.4 **测试2**
##### **9.5.4.1启动订单微服务测试，看看是否通过网关？**
我们启动80订单微服务，它从Consul注册中心通过微服务名称找到8001支付微服务进行调用，

80 → 9527 → 8001

要求访问9527网关后才能访问8001，如果我们此时启动80订单，可以做到吗？

**1 修改cloud-api-commons**

PayFeignApi接口

```plain
package com.atguigu.cloud.apis;

import com.atguigu.cloud.entities.PayDTO;
import com.atguigu.cloud.resp.ResultData;
import io.github.resilience4j.bulkhead.annotation.Bulkhead;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;

/**
 * @auther zzyy
 * @create 2023-11-09 15:29
 */
@FeignClient(value = "cloud-payment-service")
public interface PayFeignApi
{
    /**
     * 新增一条支付相关流水记录
     * @param payDTO
     * @return
     */
    @PostMapping("/pay/add")
    public ResultData addPay(@RequestBody PayDTO payDTO);

    /**
     * 按照主键记录查询支付流水信息
     * @param id
     * @return
     */
    @GetMapping("/pay/get/{id}")
    public ResultData getPayInfo(@PathVariable("id") Integer id);

    /**
     * openfeign天然支持负载均衡演示
     * @return
     */
    @GetMapping(value = "/pay/get/info")
    public String mylb();

    /**
     * Resilience4j CircuitBreaker 的例子
     * @param id
     * @return
     */
    @GetMapping(value = "/pay/circuit/{id}")
    public String myCircuit(@PathVariable("id") Integer id);

    /**
     * Resilience4j Bulkhead 的例子
     * @param id
     * @return
     */
    @GetMapping(value = "/pay/bulkhead/{id}")
    public String myBulkhead(@PathVariable("id") Integer id);

    /**
     * Resilience4j Ratelimit 的例子
     * @param id
     * @return
     */
    @GetMapping(value = "/pay/ratelimit/{id}")
    public String myRatelimit(@PathVariable("id") Integer id);


    /**
     * Micrometer(Sleuth)进行链路监控的例子
     * @param id
     * @return
     */
    @GetMapping(value = "/pay/micrometer/{id}")
    public String myMicrometer(@PathVariable("id") Integer id);

    /**
     * GateWay进行网关测试案例01
     * @param id
     * @return
     */
    @GetMapping(value = "/pay/gateway/get/{id}")
    public ResultData getById(@PathVariable("id") Integer id);

    /**
     * GateWay进行网关测试案例02
     * @return
     */
    @GetMapping(value = "/pay/gateway/info")
    public ResultData<String> getGatewayInfo();

}
```

**2 修改cloud-consumer-feign-order80**

新建OrderGateWayController

```plain
package com.atguigu.cloud.controller;

import com.atguigu.cloud.apis.PayFeignApi;
import com.atguigu.cloud.resp.ResultData;
import jakarta.annotation.Resource;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;

/**
 * @auther zzyy
 * @create 2023-11-20 16:48
 */
@RestController
public class OrderGateWayController
{
    @Resource
    private PayFeignApi payFeignApi;

    @GetMapping(value = "/feign/pay/gateway/get/{id}")
    public ResultData getById(@PathVariable("id") Integer id)
    {
        return payFeignApi.getById(id);
    }

    @GetMapping(value = "/feign/pay/gateway/info")
    public ResultData<String> getGatewayInfo()
    {
        return payFeignApi.getGatewayInfo();
    }
}
```

**3 网关开启**

测试通过

[http://localhost/feign/pay/gateway/get/1](http://localhost/feign/pay/gateway/get/1)

[http://localhost/feign/pay/gateway/info](http://localhost/feign/pay/gateway/info)

**4 网关关闭**

测试通过

[http://localhost/feign/pay/gateway/get/1](http://localhost/feign/pay/gateway/get/1)

[http://localhost/feign/pay/gateway/info](http://localhost/feign/pay/gateway/info)

**5 结论**

9527网关是否启动，毫无影响，o(╥﹏╥)o

目前的配置来看，网关被绕开了......

##### **9.5.4.2 正确做法**
**同一家公司自己人，系统内环境，直接找微服务**

```plain
@FeignClient(value = "cloud-payment-service")//自己人内部，自己访问自己，写微服务名字OK
public interface PayFeignApi
{
    /**
     * GateWay进行网关测试案例01
     * @param id
     * @return
     */
    @GetMapping(value = "/pay/gateway/get/{id}")
    public ResultData getById(@PathVariable("id") Integer id);

    /**
     * GateWay进行网关测试案例02
     * @return
     */
    @GetMapping(value = "/pay/gateway/info")
    public ResultData<String> getGatewayInfo();
}
```

**不同家公司有外人，系统外访问，先找网关再服务**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887441703-6bdfb4ff-cef3-43df-9c47-d47a8bbfb1cb.png)

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887441785-cd137085-c977-46f9-83c9-9334e2d39e86.png)

**刷新feign接口jar包**

**重启80订单微服务**

**有网关正常success**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887441893-ede0c9a5-e7d7-40fa-a8dc-b1f7c4d9f2c0.png)

**无网关异常**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887441791-f540a282-3fff-4549-8aa1-580b841d5ab5.png)

#### **9.5.5 还有问题**
请看看网关9527的yml配置，映射写死问题，^_^

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887445865-89d71fbf-9da3-48e1-b7b0-cd62a8fbb611.png)

### **9.6 GateWay高级特性**
#### **9.6.1 Route以微服务名-动态获取服务URI**
**痛点**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887445502-c4dcff8d-054b-4b72-89ec-1f61b55942dd.png)

**是什么**

![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887445407-7a0fa9d4-5004-49c9-8ba1-ea1939706a79.png)

**解决uri地址写死问题**

9527修改前YML

```plain
server:
  port: 9527

spring:
  application:
    name: cloud-gateway #以微服务注册进consul或nacos服务列表内
  cloud:
    consul: #配置consul地址
      host: localhost
      port: 8500
      discovery:
        prefer-ip-address: true
        service-name: ${spring.application.name}
    gateway:
      routes:
        - id: pay_routh1 #pay_routh1                #路由的ID(类似mysql主键ID)，没有固定规则但要求唯一，建议配合服务名
          uri: http://localhost:8001                #匹配后提供服务的路由地址
          predicates:
            - Path=/pay/gateway/get/**              # 断言，路径相匹配的进行路由

        - id: pay_routh2 #pay_routh2                #路由的ID(类似mysql主键ID)，没有固定规则但要求唯一，建议配合服务名
          uri: http://localhost:8001                #匹配后提供服务的路由地址
          predicates:
            - Path=/pay/gateway/info/**              # 断言，路径相匹配的进行路由
```

9527修改后YML

```plain
server:
  port: 9527

spring:
  application:
    name: cloud-gateway #以微服务注册进consul或nacos服务列表内
  cloud:
    consul: #配置consul地址
      host: localhost
      port: 8500
      discovery:
        prefer-ip-address: true
        service-name: ${spring.application.name}
    gateway:
      routes:
        - id: pay_routh1 #pay_routh1                #路由的ID(类似mysql主键ID)，没有固定规则但要求唯一，建议配合服务名
          #uri: http://localhost:8001                #匹配后提供服务的路由地址
          uri: lb://cloud-payment-service          #匹配后提供服务的路由地址
          predicates:
            - Path=/pay/gateway/get/**              # 断言，路径相匹配的进行路由

        - id: pay_routh2 #pay_routh2                #路由的ID(类似mysql主键ID)，没有固定规则但要求唯一，建议配合服务名
          #uri: http://localhost:8001                #匹配后提供服务的路由地址
          uri: lb://cloud-payment-service                #匹配后提供服务的路由地址
          predicates:
            - Path=/pay/gateway/info/**              # 断言，路径相匹配的进行路由
```

**测试1**

重启网关9527，80/8001保持不变

[http://localhost/feign/pay/gateway/get/1](http://localhost/feign/pay/gateway/get/1)

**测试2**

**如果将8001微服务yml文件端口修改为8007，照样访问。我实际启动的程序是main8001，但是端口名改为8007**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887445972-1aa193b8-f03f-4499-84b5-789488d27921.png)

**我们依据微服务名字，匹配查找即可**

uri：: lb://cloud-payment-service

#### **9.6.2 Predicate断言(谓词)**
##### 9.6.2.1 是什么
Route Predicate Factories这个是什么东东?

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887445387-200c5aad-4a52-43c1-83ed-834cf2ccc302.png)

##### 9.6.2.2 启动微服务gateway9527，看看IDEA后台的输出
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887454372-fc4de32e-a847-46f3-ac6b-afe2c612ea0c.png)

##### 9.6.2.3 整体架构概述
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887454375-23db6a42-4c04-4466-949f-a88e2f8b09ce.png)

##### **9.6.2.4 常用的内置Route Predicate**
**1 配置语法总体概述**

**两种配置，二选一**

1. Shortcut Configuration (快捷配置)

这是最常用的方式，因为它简洁，适合在 `.yml` 文件中快速书写。它将参数拼接成一个字符串，用逗号分隔。

**格式规范：**`name=value1,value2,value3...`

+ **特点**：参数值必须按照特定的顺序排列（顺序由对应的 Factory 类定义）。

**例子**（以 `Cookie` 断言为例）：

+ YAML

```yaml
predicates:
  - Cookie=username,zhangsan
```

这里 `Cookie` 是断言工厂的名称，`username` 是参数 1（Cookie 的 Key），`zhangsan` 是参数 2（正则匹配的 Value）。

---

2. Fully Expanded Arguments (完全展开参数)

这种方式采用标准的 YAML 键值对（Key-Value）结构。它更具可读性，且不需要记住参数的先后顺序。

**格式规范：** 使用 `name` 属性指定工厂名称，使用 `args` 属性指定具体的参数名和值。

+ **特点**：结构清晰，像是在写一个 JSON 对象。

**例子**（同样是上面的 `Cookie` 断言）：

```yaml
predicates:
  - name: Cookie
    args:
      name: username
      regexp: zhangsan
```



**3 常用断言api**

_#id：我们自定义的路由 ID，保持唯一_  
_##uri：目标服务地址_  
_##predicates：路由条件，Predicate接受一个输入参数返回一个布尔值。_  
_##            该属性包含多种默认方法来将Predicate组合成其他复杂的逻辑(比如：与，或，非)_

**1 After Route Predicate**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887457893-694d84f7-e894-4263-9bad-4c979202deff.png)

**如何获得ZonedDateTime**

```plain
import java.time.ZonedDateTime;
/**
 * ZonedDateTime 示例类
 * 用于演示 Java 8 中 ZonedDateTime 类的使用
 * ZonedDateTime 包含时区信息的日期时间类
 * @author zzyy
 * @version 1.0
 * @date 2023/11/20
 */
public class ZonedDateTimeDemo {
    /**
     * 主方法，程序入口
     * 演示获取当前带时区的日期时间
     * @param args 命令行参数
     */
    public static void main(String[] args) {
        // 获取当前默认时区的日期时间对象
        // ZonedDateTime.now() 会获取系统默认时区的当前时间
        ZonedDateTime zbj = ZonedDateTime.now();  // 默认时区
        // 输出当前日期时间信息
        // 输出格式示例: 2023-11-20T15:30:45.123+08:00[Asia/Shanghai]
        System.out.println("当前默认时区时间: " + zbj);
        // 输出说明：
        // 格式包含：年-月-日T时:分:秒.毫秒+时区[时区名称]
        // 例如：2023-11-20T15:30:45.123+08:00[Asia/Shanghai] 表示北京时间
    }
}
```

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887457846-8aca4007-470a-473a-b63b-18f99abd0bac.png) 

**YML**

```plain
server:
  port: 9527

spring:
  application:
    name: cloud-gateway #以微服务注册进consul或nacos服务列表内
  cloud:
    consul: #配置consul地址
      host: localhost
      port: 8500
      discovery:
        prefer-ip-address: true
        service-name: ${spring.application.name}
    gateway:
      routes:
        - id: pay_routh1 #pay_routh1                #路由的ID(类似mysql主键ID)，没有固定规则但要求唯一，建议配合服务名
          #uri: http://localhost:8001                #匹配后提供服务的路由地址
          uri: lb://cloud-payment-service          #匹配后提供服务的路由地址
          predicates:
            - Path=/pay/gateway/get/**              # 断言，路径相匹配的进行路由
            - After=2023-11-20T17:38:13.586918800+08:00[Asia/Shanghai]

        - id: pay_routh2 #pay_routh2                #路由的ID(类似mysql主键ID)，没有固定规则但要求唯一，建议配合服务名
          #uri: http://localhost:8001                #匹配后提供服务的路由地址
          uri: lb://cloud-payment-service                #匹配后提供服务的路由地址
          predicates:
            - Path=/pay/gateway/info/**              # 断言，路径相匹配的进行路由
```

**2 Before Route Predicate(家庭作业)**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887460272-6aec0134-bd1a-49e2-933c-febf455dedce.png)

YML

```plain
server:
  port: 9527

spring:
  application:
    name: cloud-gateway #以微服务注册进consul或nacos服务列表内
  cloud:
    consul: #配置consul地址
      host: localhost
      port: 8500
      discovery:
        prefer-ip-address: true
        service-name: ${spring.application.name}
    gateway:
      routes:
        - id: pay_routh1 #pay_routh1                #路由的ID(类似mysql主键ID)，没有固定规则但要求唯一，建议配合服务名
          #uri: http://localhost:8001                #匹配后提供服务的路由地址
          uri: lb://cloud-payment-service          #匹配后提供服务的路由地址
          predicates:
            - Path=/pay/gateway/get/**              # 断言，路径相匹配的进行路由
            #- After=2023-11-20T17:38:13.586918800+08:00[Asia/Shanghai]
            - Before=2023-11-27T15:25:06.424566300+08:00[Asia/Shanghai] #超过规定时间不可访问

        - id: pay_routh2 #pay_routh2                #路由的ID(类似mysql主键ID)，没有固定规则但要求唯一，建议配合服务名
          #uri: http://localhost:8001                #匹配后提供服务的路由地址
          uri: lb://cloud-payment-service                #匹配后提供服务的路由地址
          predicates:
            - Path=/pay/gateway/info/**              # 断言，路径相匹配的进行路由
```

**3 Between Route Predicate(家庭作业)**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887460324-984465d1-f1b4-4bff-826b-992651407422.png)

YML

```plain
server:
  port: 9527

spring:
  application:
    name: cloud-gateway #以微服务注册进consul或nacos服务列表内
  cloud:
    consul: #配置consul地址
      host: localhost
      port: 8500
      discovery:
        prefer-ip-address: true
        service-name: ${spring.application.name}
    gateway:
      routes:
        - id: pay_routh1 #pay_routh1                #路由的ID(类似mysql主键ID)，没有固定规则但要求唯一，建议配合服务名
          #uri: http://localhost:8001                #匹配后提供服务的路由地址
          uri: lb://cloud-payment-service          #匹配后提供服务的路由地址
          predicates:
            - Path=/pay/gateway/get/**              # 断言，路径相匹配的进行路由
            #- After=2023-11-20T17:38:13.586918800+08:00[Asia/Shanghai]
            #- Before=2023-11-20T17:58:13.586918800+08:00[Asia/Shanghai]
            - Between=2023-11-21T17:38:13.586918800+08:00[Asia/Shanghai],2023-11-22T17:38:13.586918800+08:00[Asia/Shanghai]

        - id: pay_routh2 #pay_routh2                #路由的ID(类似mysql主键ID)，没有固定规则但要求唯一，建议配合服务名
          #uri: http://localhost:8001                #匹配后提供服务的路由地址
          uri: lb://cloud-payment-service                #匹配后提供服务的路由地址
          predicates:
            - Path=/pay/gateway/info/**              # 断言，路径相匹配的进行路由
```

**4 Cookie Route Predicate**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887504876-e980dd65-0d79-4583-9edc-03ec55df9b47.png)

| Cookie Route Predicate需要两个参数，一个是 Cookie name ,一个是正则表达式。<br/>路由规则会通过获取对应的 Cookie name 值和正则表达式去匹配，如果匹配上就会执行路由，如果没有匹配上则不执行 |
| :--- |


**YML **

```plain
server:
  port: 9527

spring:
  application:
    name: cloud-gateway #以微服务注册进consul或nacos服务列表内
  cloud:
    consul: #配置consul地址
      host: localhost
      port: 8500
      discovery:
        prefer-ip-address: true
        service-name: ${spring.application.name}
    gateway:
      routes:
        - id: pay_routh1 #pay_routh1                #路由的ID(类似mysql主键ID)，没有固定规则但要求唯一，建议配合服务名
          #uri: http://localhost:8001                #匹配后提供服务的路由地址
          uri: lb://cloud-payment-service          #匹配后提供服务的路由地址
          predicates:
            - Path=/pay/gateway/get/**              # 断言，路径相匹配的进行路由
            #- After=2023-11-20T17:38:13.586918800+08:00[Asia/Shanghai]
            - Before=2023-12-29T17:58:13.586918800+08:00[Asia/Shanghai]
            #- Between=2023-11-21T17:38:13.586918800+08:00[Asia/Shanghai],2023-11-22T17:38:13.586918800+08:00[Asia/Shanghai]
            - Cookie=username,zzyy

        - id: pay_routh2 #pay_routh2                #路由的ID(类似mysql主键ID)，没有固定规则但要求唯一，建议配合服务名
          #uri: http://localhost:8001                #匹配后提供服务的路由地址
          uri: lb://cloud-payment-service                #匹配后提供服务的路由地址
          predicates:
            - Path=/pay/gateway/info/**              # 断言，路径相匹配的进行路由
```

**方法1，原生命令**

**不带cookie参数   curl http://localhost:9527/pay/gateway/get/1**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887504797-01c8da1e-8f4e-4e58-a04b-e4e0971e871b.png)

**自带cookie参数   curl http://localhost:9527/pay/gateway/get/1 --cookie "username=zzyy"**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887504785-6ec6ca4f-b9fd-46da-b96a-7ada7ee02287.png)

**方法2，postman**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887504870-217e91c9-a305-4a93-a9c8-28cd3ce69a44.png)

**方法3，chrome浏览器**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887505048-dfd2aa05-c27e-4070-9bc7-bfa4c06266d3.png)

**5 Header Route Predicate**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887510031-bdee5291-d99b-4652-839e-0379af9f2e73.png)

两个参数：一个是属性名称和一个正则表达式，这个属性值和正则表达式匹配则执行。

**YML**

```plain
server:
  port: 9527

spring:
  application:
    name: cloud-gateway #以微服务注册进consul或nacos服务列表内
  cloud:
    consul: #配置consul地址
      host: localhost
      port: 8500
      discovery:
        prefer-ip-address: true
        service-name: ${spring.application.name}
    gateway:
      routes:
        - id: pay_routh1 #pay_routh1                #路由的ID(类似mysql主键ID)，没有固定规则但要求唯一，建议配合服务名
          #uri: http://localhost:8001                #匹配后提供服务的路由地址
          uri: lb://cloud-payment-service          #匹配后提供服务的路由地址
          predicates:
            - Path=/pay/gateway/get/**              # 断言，路径相匹配的进行路由
            #- After=2023-11-20T17:38:13.586918800+08:00[Asia/Shanghai]
            - Before=2023-12-29T17:58:13.586918800+08:00[Asia/Shanghai]
            #- Between=2023-11-21T17:38:13.586918800+08:00[Asia/Shanghai],2023-11-22T17:38:13.586918800+08:00[Asia/Shanghai]
            #- Cookie=username,zzyy
            - Header=X-Request-Id, \d+  # 请求头要有X-Request-Id属性并且值为整数的正则表达式

        - id: pay_routh2 #pay_routh2                #路由的ID(类似mysql主键ID)，没有固定规则但要求唯一，建议配合服务名
          #uri: http://localhost:8001                #匹配后提供服务的路由地址
          uri: lb://cloud-payment-service                #匹配后提供服务的路由地址
          predicates:
            - Path=/pay/gateway/info/**              # 断言，路径相匹配的进行路由
```

**方法1，原生命令**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887512591-f2243393-ee27-49c8-86a9-378aabc4fe12.png)

curl http://localhost:9527/pay/gateway/get/1 -H  "X-Request-Id:123456"

curl http://localhost:9527/pay/gateway/get/1 -H  "X-Request-Id:abcd"

**方法2，postman**

上图正确，下图错误验证

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887513636-e92f5819-d519-4dc9-9001-5808456b1265.png)

| ================================================================================ |
| :--- |


<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887513536-cebd1a29-4b87-4a34-a235-e8451631bd39.png)

**6 Host Route Predicate**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887513550-842ebcac-de59-4a42-812f-13ae09b24117.png)

Host Route Predicate 接收一组参数，一组匹配的域名列表，这个模板是一个 ant 分隔的模板，用.号作为分隔符。

它通过参数中的主机地址作为匹配规则。

**YML**

```plain
server:
  port: 9527

spring:
  application:
    name: cloud-gateway #以微服务注册进consul或nacos服务列表内
  cloud:
    consul: #配置consul地址
      host: localhost
      port: 8500
      discovery:
        prefer-ip-address: true
        service-name: ${spring.application.name}
    gateway:
      routes:
        - id: pay_routh1 #pay_routh1                #路由的ID(类似mysql主键ID)，没有固定规则但要求唯一，建议配合服务名
          #uri: http://localhost:8001                #匹配后提供服务的路由地址
          uri: lb://cloud-payment-service          #匹配后提供服务的路由地址
          predicates:
            - Path=/pay/gateway/get/**              # 断言，路径相匹配的进行路由
            #- After=2023-11-20T17:38:13.586918800+08:00[Asia/Shanghai]
            - Before=2023-12-29T17:58:13.586918800+08:00[Asia/Shanghai]
            #- Between=2023-11-21T17:38:13.586918800+08:00[Asia/Shanghai],2023-11-22T17:38:13.586918800+08:00[Asia/Shanghai]
            #- Cookie=username,zzyy
            #- Header=X-Request-Id, \d+  # 请求头要有X-Request-Id属性并且值为整数的正则表达式
            - Host=**.atguigu.com

        - id: pay_routh2 #pay_routh2                #路由的ID(类似mysql主键ID)，没有固定规则但要求唯一，建议配合服务名
          #uri: http://localhost:8001                #匹配后提供服务的路由地址
          uri: lb://cloud-payment-service                #匹配后提供服务的路由地址
          predicates:
            - Path=/pay/gateway/info/**              # 断言，路径相匹配的进行路由
```

**方法1，原生命令**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887515172-a1b90c98-aa46-4d94-b666-a74c269247a8.png)

正确：     curl http://localhost:9527/pay/gateway/get/3 -H  "Host:www.atguigu.com"

正确：     curl http://localhost:9527/pay/gateway/get/3 -H  "Host:java.atguigu.com"

错误：     curl http://localhost:9527/pay/gateway/get/3 -H  "Host:java.atguigu.net"

**方法2，postman**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887517200-fa5d1607-f509-45dc-bf21-88ff1eb6f483.png)

**7 Path Route Predicate**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887518519-490cb335-77f5-4128-85cc-daf05e621a2d.png)

**YML**

```plain
server:
  port: 9527

spring:
  application:
    name: cloud-gateway #以微服务注册进consul或nacos服务列表内
  cloud:
    consul: #配置consul地址
      host: localhost
      port: 8500
      discovery:
        prefer-ip-address: true
        service-name: ${spring.application.name}
    gateway:
      routes:
        - id: pay_routh1 #pay_routh1                #路由的ID(类似mysql主键ID)，没有固定规则但要求唯一，建议配合服务名
          #uri: http://localhost:8001                #匹配后提供服务的路由地址
          uri: lb://cloud-payment-service          #匹配后提供服务的路由地址
          predicates:
            - Path=/pay/gateway/get/**              # 断言，路径相匹配的进行路由
            #- After=2023-11-20T17:38:13.586918800+08:00[Asia/Shanghai]
            - Before=2023-12-29T17:58:13.586918800+08:00[Asia/Shanghai]
            #- Between=2023-11-21T17:38:13.586918800+08:00[Asia/Shanghai],2023-11-22T17:38:13.586918800+08:00[Asia/Shanghai]
            #- Cookie=username,zzyy
            #- Header=X-Request-Id, \d+  # 请求头要有X-Request-Id属性并且值为整数的正则表达式
            - Host=**.atguigu.com

        - id: pay_routh2 #pay_routh2                #路由的ID(类似mysql主键ID)，没有固定规则但要求唯一，建议配合服务名
          #uri: http://localhost:8001                #匹配后提供服务的路由地址
          uri: lb://cloud-payment-service                #匹配后提供服务的路由地址
          predicates:
            - Path=/pay/gateway/info/**              # 断言，路径相匹配的进行路由
```

**8 Query Route Predicate**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887520289-d7a48c80-5244-469b-8544-d9315aa819b5.png)

支持传入两个参数，一个是属性名，一个为属性值，属性值可以是正则表达式。

**YML**

```plain
server:
  port: 9527

spring:
  application:
    name: cloud-gateway #以微服务注册进consul或nacos服务列表内
  cloud:
    consul: #配置consul地址
      host: localhost
      port: 8500
      discovery:
        prefer-ip-address: true
        service-name: ${spring.application.name}
    gateway:
      routes:
        - id: pay_routh1 #pay_routh1                #路由的ID(类似mysql主键ID)，没有固定规则但要求唯一，建议配合服务名
          #uri: http://localhost:8001                #匹配后提供服务的路由地址
          uri: lb://cloud-payment-service          #匹配后提供服务的路由地址
          predicates:
            - Path=/pay/gateway/get/**              # 断言，路径相匹配的进行路由
            #- After=2023-11-20T17:38:13.586918800+08:00[Asia/Shanghai]
            - Before=2023-12-29T17:58:13.586918800+08:00[Asia/Shanghai]
            #- Between=2023-11-21T17:38:13.586918800+08:00[Asia/Shanghai],2023-11-22T17:38:13.586918800+08:00[Asia/Shanghai]
            #- Cookie=username,zzyy
            #- Header=X-Request-Id, \d+  # 请求头要有X-Request-Id属性并且值为整数的正则表达式
            #- Host=**.atguigu.com
            - Query=username, \d+  # 要有参数名username并且值还要是整数才能路由

        - id: pay_routh2 #pay_routh2                #路由的ID(类似mysql主键ID)，没有固定规则但要求唯一，建议配合服务名
          #uri: http://localhost:8001                #匹配后提供服务的路由地址
          uri: lb://cloud-payment-service                #匹配后提供服务的路由地址
          predicates:
            - Path=/pay/gateway/info/**              # 断言，路径相匹配的进行路由
```

**测试**

**http://localhost:9527/pay/gateway/get/3?username=123**

_**http://localhost:9527/pay/gateway/get/3?username=abc 要有参数名username并且值还要是整数才能路由**_

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887520440-9a9fadb6-2265-432f-925a-c2c255521470.png)

**9 RemoteAddr route predicate**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887521553-63ae3020-04cb-4b36-8e29-d6dc800ad7a5.png)

**YML**

```plain
server:
  port: 9527

spring:
  application:
    name: cloud-gateway #以微服务注册进consul或nacos服务列表内
  cloud:
    consul: #配置consul地址
      host: localhost
      port: 8500
      discovery:
        prefer-ip-address: true
        service-name: ${spring.application.name}
    gateway:
      routes:
        - id: pay_routh1 #pay_routh1                #路由的ID(类似mysql主键ID)，没有固定规则但要求唯一，建议配合服务名
          #uri: http://localhost:8001                #匹配后提供服务的路由地址
          uri: lb://cloud-payment-service          #匹配后提供服务的路由地址
          predicates:
            - Path=/pay/gateway/get/**              # 断言，路径相匹配的进行路由
            #- After=2023-11-20T17:38:13.586918800+08:00[Asia/Shanghai]
            - Before=2023-12-29T17:58:13.586918800+08:00[Asia/Shanghai]
            #- Between=2023-11-21T17:38:13.586918800+08:00[Asia/Shanghai],2023-11-22T17:38:13.586918800+08:00[Asia/Shanghai]
            #- Cookie=username,zzyy
            #- Header=X-Request-Id, \d+  # 请求头要有X-Request-Id属性并且值为整数的正则表达式
            #- Host=**.atguigu.com
            #- Query=username, \d+  # 要有参数名username并且值还要是整数才能路由
            - RemoteAddr=192.168.124.1/24 # 外部访问我的IP限制，最大跨度不超过32，目前是1~24它们是 CIDR 表示法。

        - id: pay_routh2 #pay_routh2                #路由的ID(类似mysql主键ID)，没有固定规则但要求唯一，建议配合服务名
          #uri: http://localhost:8001                #匹配后提供服务的路由地址
          uri: lb://cloud-payment-service                #匹配后提供服务的路由地址
          predicates:
            - Path=/pay/gateway/info/**              # 断言，路径相匹配的进行路由
```

**CIDR网络IP划分(无类别域间路由Classless Inter-Domain Routing缩写)**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887521496-58252d40-27b2-41b3-9f68-ea76cf8f6ea0.png)

**10 Method Route Predicate(家庭作业)**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887525453-1e818f7b-1c99-4e8e-9f05-476042854911.png)

配置某个请求地址，只能用Get/Post方法访问，方法限制

**4 上述配置小总结**

**All**

```plain
server:
  port: 9527

spring:
  application:
    name: cloud-gateway #以微服务注册进consul或nacos服务列表内
  cloud:
    consul: #配置consul地址
      host: localhost
      port: 8500
      discovery:
        prefer-ip-address: true
        service-name: ${spring.application.name}
    gateway:
      routes:
        - id: pay_routh1 #pay_routh1                #路由的ID(类似mysql主键ID)，没有固定规则但要求唯一，建议配合服务名
          #uri: http://localhost:8001                #匹配后提供服务的路由地址
          uri: lb://cloud-payment-service                #匹配后提供服务的路由地址
          predicates:
            - Path=/pay/gateway/get/**              # 断言，路径相匹配的进行路由
            - After=2023-12-30T23:02:39.079979400+08:00[Asia/Shanghai]
            #- Cookie=username,zzyy
            # - Header=X-Request-Id, \d+ # 请求头要有X-Request-Id属性并且值为整数的正则表达式
            #- Host=**.atguigu.com
            #- Query=username, \d+  # 要有参数名username并且值还要是整数才能路由
            #- RemoteAddr=192.168.124.1/24 # 外部访问我的IP限制，最大跨度不超过32，目前是1~24它们是 CIDR 表示法。

        - id: pay_routh2 #pay_routh2                #路由的ID(类似mysql主键ID)，没有固定规则但要求唯一，建议配合服务名
          #uri: http://localhost:8001                #匹配后提供服务的路由地址
          uri: lb://cloud-payment-service
          predicates:
            - Path=/pay/gateway/info/**              # 断言，路径相匹配的进行路由
```

**Predicate就是为了实现一组匹配规则，让请求过来找到对应的Route进行处理。**

##### **9.6.2.5 自定义断言，XXXRoutePredicateFactory规则**
**痛点**

**原有的断言配置不满足业务怎么办？**

**看看AfterRoutePredicateFactory**

源代码：

```java
public abstract class AbstractRoutePredicateFactory<C> extends AbstractConfigurable<C> implements RoutePredicateFactory<C> {
    public AbstractRoutePredicateFactory(Class<C> configClass) 
    {
        super(configClass);
    }
}
```

**架构概述**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887527935-8a0f84cd-9159-4a7d-8256-ed8474a6f343.png)

**模板套路**

要么继承AbstractRoutePredicateFactory抽象类

要么实现RoutePredicateFactory接口

开头任意取名，但是必须以RoutePredicateFactory后缀结尾

**自定义路由断言规则步骤套路**

需求说明：自定义配置会员等级userTpye，按照钻/金/银和yml配置的会员等级，以适配是否可以访问

**编写步骤**

**1 新建类名XXX需要以RoutePredicateFactory结尾并继承AbstractRoutePredicateFactory类**

```java
@Component //标注不可忘
public class MyRoutePredicateFactory extends AbstractRoutePredicateFactory<MyRoutePredicateFactory.Config>{

}
```

**2 重写apply方法**

```java
@Override
public Predicate<ServerWebExchange> apply(MyRoutePredicateFactory.Config config){
    return null;
}
```

**3 新建apply方法所需要的静态内部类MyRoutePredicateFactory.Config，****这个Config类就是我们的路由断言规则，重要**

```java
//这个Config类就是我们的路由断言规则，重要
@Validated
public static class Config{
    @Setter
    @Getter
    @NotEmpty
    private String userType; //钻、金、银等用户等级
}
```

**4 空参构造方法，内部调用super**

```java
public MyRoutePredicateFactory(){
    super(MyRoutePredicateFactory.Config.class);
}
```

**5 重写apply方法第二版**

```java
@Override
public Predicate<ServerWebExchange> apply(MyRoutePredicateFactory.Config config){
    return new Predicate<ServerWebExchange>(){
        @Override
        public boolean test(ServerWebExchange serverWebExchange){
            //检查request的参数里面，userType是否为指定的值，符合配置就通过
            String userType = serverWebExchange.getRequest().getQueryParams().getFirst("userType");

            if (userType == null) return false;

            //如果说参数存在，就和config的数据进行比较
            if(userType.equals(config.getUserType())) {
                return true;
            }

            return false;
        }
    };
}
```

**完整代码V1**

```java
package com.atguigu.cloud.mygateway;

import jakarta.validation.constraints.NotEmpty;
import lombok.Getter;
import lombok.Setter;
import org.springframework.cloud.gateway.handler.predicate.AbstractRoutePredicateFactory;
import org.springframework.stereotype.Component;
import org.springframework.validation.annotation.Validated;
import org.springframework.web.server.ServerWebExchange;

import java.util.function.Predicate;

@Component
public class MyRoutePredicateFactory extends AbstractRoutePredicateFactory<MyRoutePredicateFactory.Config>
{
    public MyRoutePredicateFactory()
    {
        super(MyRoutePredicateFactory.Config.class);
    }

    @Validated
    public static class Config{
        @Setter
        @Getter
        @NotEmpty
        private String userType; //钻、金、银等用户等级
    }

    @Override
    public Predicate<ServerWebExchange> apply(MyRoutePredicateFactory.Config config)
    {
        return new Predicate<ServerWebExchange>()
        {
            @Override
            public boolean test(ServerWebExchange serverWebExchange)
            {
                //检查request的参数里面，userType是否为指定的值，符合配置就通过
                String userType = serverWebExchange.getRequest().getQueryParams().getFirst("userType");

                if (userType == null) return false;

                //如果说参数存在，就和config的数据进行比较
                if(userType.equals(config.getUserType())) {
                    return true;
                }

                return false;
            }
        };
    }
}
```

**测试1**

**YML**

```yaml
server:
  port: 9527

spring:
  application:
    name: cloud-gateway #以微服务注册进consul或nacos服务列表内
  cloud:
    consul: #配置consul地址
      host: localhost
      port: 8500
      discovery:
        prefer-ip-address: true
        service-name: ${spring.application.name}
    gateway:
      routes:
        - id: pay_routh1 #pay_routh1                #路由的ID(类似mysql主键ID)，没有固定规则但要求唯一，建议配合服务名
          #uri: http://localhost:8001                #匹配后提供服务的路由地址
          uri: lb://cloud-payment-service                #匹配后提供服务的路由地址
          predicates:
            - Path=/pay/gateway/get/**              # 断言，路径相匹配的进行路由
            - After=2023-12-30T23:02:39.079979400+08:00[Asia/Shanghai]
            #- Cookie=username,zzyy
            #- Header=X-Request-Id, \d+ # 请求头要有X-Request-Id属性并且值为整数的正则表达式
            #- Host=**.atguigu.com
            #- Query=username, \d+  # 要有参数名username并且值还要是整数才能路由
            #- RemoteAddr=192.168.124.1/24 # 外部访问我的IP限制，最大跨度不超过32，目前是1~24它们是 CIDR 表示法。
            - My=diamond

        - id: pay_routh2 #pay_routh2                #路由的ID(类似mysql主键ID)，没有固定规则但要求唯一，建议配合服务名
          #uri: http://localhost:8001                #匹配后提供服务的路由地址
          uri: lb://cloud-payment-service
          predicates:
            - Path=/pay/gateway/info/**              # 断言，路径相匹配的进行路由
```

**启动后？？？**

**故障现象**

org.springframework.boot.context.properties.bind.BindException: Failed to bind properties under '' to com.atguigu.cloud.mygateway.MyRoutePredicateFactory$Config

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887528683-1999b5bb-3b6d-492e-b61e-4a16267cff53.png)

Caused by: org.springframework.boot.context.properties.bind.validation.BindValidationException: Binding validation errors on 

**导致原因**

为什么Shortcut Configuration不生效？

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887528720-a62e8c33-5ce5-4a08-9fb3-c30db0efedb9.png)

**解决方案**

先解决问题，让我们自定义的能用

Fully Expanded Arguments

YML

```yaml
server:
  port: 9527

spring:
  application:
    name: cloud-gateway #以微服务注册进consul或nacos服务列表内
  cloud:
    consul: #配置consul地址
      host: localhost
      port: 8500
      discovery:
        prefer-ip-address: true
        service-name: ${spring.application.name}
    gateway:
      routes:
        - id: pay_routh1 #pay_routh1                #路由的ID(类似mysql主键ID)，没有固定规则但要求唯一，建议配合服务名
          #uri: http://localhost:8001                #匹配后提供服务的路由地址
          uri: lb://cloud-payment-service                #匹配后提供服务的路由地址
          predicates:
            - Path=/pay/gateway/get/**              # 断言，路径相匹配的进行路由
            - After=2023-12-30T23:02:39.079979400+08:00[Asia/Shanghai]
            #- Cookie=username,zzyy
            # - Header=X-Request-Id, \d+ # 请求头要有X-Request-Id属性并且值为整数的正则表达式
            #- Host=**.atguigu.com
            #- Query=username, \d+  # 要有参数名username并且值还要是整数才能路由
            #- RemoteAddr=192.168.124.1/24 # 外部访问我的IP限制，最大跨度不超过32，目前是1~24它们是 CIDR 表示法。
            #- My=diamond
            - name: My
              args:
                userType: diamond

        - id: pay_routh2 #pay_routh2                #路由的ID(类似mysql主键ID)，没有固定规则但要求唯一，建议配合服务名
          #uri: http://localhost:8001                #匹配后提供服务的路由地址
          uri: lb://cloud-payment-service
          predicates:
            - Path=/pay/gateway/info/**              # 断言，路径相匹配的进行路由
```

success

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887531690-8ba45e46-95c1-4d0e-b375-ae2f87c53d0b.png)

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887532618-ca6249a3-afc8-44c9-afec-fd3d68dd23f0.png)

**bug分析**

缺少shortcutFieldOrder方法的实现，所以不支持短格式

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887534764-7c8fdb9c-1f74-45a3-93fd-17957d2ee403.png)

**测试2**

**完整代码02**

```plain
package com.atguigu.cloud.mygateway;

import jakarta.validation.constraints.NotEmpty;
import lombok.Getter;
import lombok.Setter;
import org.springframework.cloud.gateway.handler.predicate.AbstractRoutePredicateFactory;
import org.springframework.stereotype.Component;
import org.springframework.validation.annotation.Validated;
import org.springframework.web.server.ServerWebExchange;

import java.util.ArrayList;
import java.util.List;
import java.util.function.Predicate;

/**
 * @auther zzyy
 * @create 2023-04-23 18:30
 */
@Component
public class MyRoutePredicateFactory extends AbstractRoutePredicateFactory<MyRoutePredicateFactory.Config>
{
    public MyRoutePredicateFactory()
    {
        super(MyRoutePredicateFactory.Config.class);
    }

    @Validated
    public static class Config{
        @Setter
        @Getter
        @NotEmpty
        private String userType; //钻、金、银等用户等级
    }

    @Override
    public Predicate<ServerWebExchange> apply(MyRoutePredicateFactory.Config config)
    {
        return new Predicate<ServerWebExchange>()
        {
            @Override
            public boolean test(ServerWebExchange serverWebExchange)
            {
                //检查request的参数里面，userType是否为指定的值，符合配置就通过
                String userType = serverWebExchange.getRequest().getQueryParams().getFirst("userType");

                if (userType == null) return false;

                //如果说参数存在，就和config的数据进行比较
                if(userType.equals(config.getUserType())) {
                    return true;
                }

                return false;
            }
        };
    }

@Override
public List<String> shortcutFieldOrder() {
  return Collections.singletonList("userType");
}


}
```

**YML**

```plain
server:
  port: 9527

spring:
  application:
    name: cloud-gateway #以微服务注册进consul或nacos服务列表内
  cloud:
    consul: #配置consul地址
      host: localhost
      port: 8500
      discovery:
        prefer-ip-address: true
        service-name: ${spring.application.name}
    gateway:
      routes:
        - id: pay_routh1 #pay_routh1                #路由的ID(类似mysql主键ID)，没有固定规则但要求唯一，建议配合服务名
          #uri: http://localhost:8001                #匹配后提供服务的路由地址
          uri: lb://cloud-payment-service                #匹配后提供服务的路由地址
          predicates:
            - Path=/pay/gateway/get/**              # 断言，路径相匹配的进行路由
            - After=2023-12-30T23:02:39.079979400+08:00[Asia/Shanghai]
            #- Cookie=username,zzyy
            # - Header=X-Request-Id, \d+ # 请求头要有X-Request-Id属性并且值为整数的正则表达式
            #- Host=**.atguigu.com
            #- Query=username, \d+  # 要有参数名username并且值还要是整数才能路由
            #- RemoteAddr=192.168.124.1/24 # 外部访问我的IP限制，最大跨度不超过32，目前是1~24它们是 CIDR 表示法。
            - My=diamond
            #- name: My
            #  args:
            #    userType: diamond

        - id: pay_routh2 #pay_routh2                #路由的ID(类似mysql主键ID)，没有固定规则但要求唯一，建议配合服务名
          #uri: http://localhost:8001                #匹配后提供服务的路由地址
          uri: lb://cloud-payment-service
          predicates:
            - Path=/pay/gateway/info/**              # 断言，路径相匹配的进行路由
```

#### **9.6.3 Filter过滤**
##### **9.6.3.1 **概述
**一句话**

**SpringMVC里面的的拦截器Interceptor，Servlet的过滤器**

**“pre”和 “post” 分别会在请求被执行前调用和被执行后调用，用来修改请求和响应信息**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887609770-5697ce2f-677e-47d5-b7de-1438630f99fd.png)

**能干嘛**

请求鉴权

异常处理

记录接口调用时长统计，重点，大厂面试设计题

**类型**

**1 全局默认过滤器Global Filters**

gateway出厂默认已有的，直接用即可，主要作用于所有的路由

不需要在配置文件中配置，作用在所有的路由上，实现GlobalFilter接口即可

**2 单一内置过滤器GatewayFilter**

也可以称为网关过滤器，这种过滤器主要是作用于单一路由或者某个路由分组

**3 自定义过滤器**

##### **9.6.3.2 **Gateway内置的过滤器
###### 1 请求头(RequestHeader)相关组
**6.1. The AddRequestHeader GatewayFilter Factory 指定请求头内容ByName**

**8001微服务PayGateWayController新增方法**

```plain
@RestController
public class PayGateWayController
{
    @Resource
    PayService payService;

    @GetMapping(value = "/pay/gateway/get/{id}")
    public ResultData<Pay> getById(@PathVariable("id") Integer id)
    {
        Pay pay = payService.getById(id);
        return ResultData.success(pay);
    }

    @GetMapping(value = "/pay/gateway/info")
    public ResultData<String> getGatewayInfo()
    {
        return ResultData.success("gateway info test："+ IdUtil.simpleUUID());
    }

    @GetMapping(value = "/pay/gateway/filter")
    public ResultData<String> getGatewayFilter(HttpServletRequest request)
    {
        String result = "";
        Enumeration<String> headers = request.getHeaderNames();
        while(headers.hasMoreElements())
        {
            String headName = headers.nextElement();
            String headValue = request.getHeader(headName);
            System.out.println("请求头名: " + headName +"\t\t\t"+"请求头值: " + headValue);
            if(headName.equalsIgnoreCase("X-Request-atguigu1")
                    || headName.equalsIgnoreCase("X-Request-atguigu2")) {
                result = result+headName + "\t " + headValue +" ";
            }
        }
        return ResultData.success("getGatewayFilter 过滤器 test： "+result+" \t "+ DateUtil.now());
    }
}
```

**9527网关YML添加过滤内容**

```yaml
server:
  port: 9527

spring:
  application:
    name: cloud-gateway #以微服务注册进consul或nacos服务列表内
  cloud:
    consul: #配置consul地址
      host: localhost
      port: 8500
      discovery:
        prefer-ip-address: true
        service-name: ${spring.application.name}
    gateway:
      routes:
        - id: pay_routh1 #pay_routh1                #路由的ID(类似mysql主键ID)，没有固定规则但要求唯一，建议配合服务名
          #uri: http://localhost:8001                #匹配后提供服务的路由地址
          uri: lb://cloud-payment-service                #匹配后提供服务的路由地址
          predicates:
            - Path=/pay/gateway/get/**              # 断言，路径相匹配的进行路由
            - After=2023-12-30T23:02:39.079979400+08:00[Asia/Shanghai]
            #- Cookie=username,zzyy
            # - Header=X-Request-Id, \d+ # 请求头要有X-Request-Id属性并且值为整数的正则表达式
            #- Host=**.atguigu.com
            #- Query=username, \d+  # 要有参数名username并且值还要是整数才能路由
            #- RemoteAddr=192.168.124.1/24 # 外部访问我的IP限制，最大跨度不超过32，目前是1~24它们是 CIDR 表示法。
            - My=gold
#            - name: My
#              args:
#                userType: diamond

        - id: pay_routh2 #pay_routh2                #路由的ID(类似mysql主键ID)，没有固定规则但要求唯一，建议配合服务名
          #uri: http://localhost:8001                #匹配后提供服务的路由地址
          uri: lb://cloud-payment-service
          predicates:
            - Path=/pay/gateway/info/**              # 断言，路径相匹配的进行路由

        - id: pay_routh3 #pay_routh3
          uri: lb://cloud-payment-service                #匹配后提供服务的路由地址
          predicates:
            - Path=/pay/gateway/filter/**              # 断言，路径相匹配的进行路由
          filters:
            - AddRequestHeader=X-Request-atguigu1,atguiguValue1  # 请求头kv，若一头含有多参则重写一行设置
            - AddRequestHeader=X-Request-atguigu2,atguiguValue2
```

**6.18. The RemoveRequestHeader GatewayFilter Factory 删除请求头ByName**

**修改前**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887609720-436e5829-f7df-4522-8237-1e08a3c119e6.png)

**YML**

```plain
server:
  port: 9527

spring:
  application:
    name: cloud-gateway #以微服务注册进consul或nacos服务列表内
  cloud:
    consul: #配置consul地址
      host: localhost
      port: 8500
      discovery:
        prefer-ip-address: true
        service-name: ${spring.application.name}
    gateway:
      routes:
        - id: pay_routh1 #pay_routh1                #路由的ID(类似mysql主键ID)，没有固定规则但要求唯一，建议配合服务名
          #uri: http://localhost:8001                #匹配后提供服务的路由地址
          uri: lb://cloud-payment-service                #匹配后提供服务的路由地址
          predicates:
            - Path=/pay/gateway/get/**              # 断言，路径相匹配的进行路由
            - After=2023-12-30T23:02:39.079979400+08:00[Asia/Shanghai]
            #- Cookie=username,zzyy
            # - Header=X-Request-Id, \d+ # 请求头要有X-Request-Id属性并且值为整数的正则表达式
            #- Host=**.atguigu.com
            #- Query=username, \d+  # 要有参数名username并且值还要是整数才能路由
            #- RemoteAddr=192.168.124.1/24 # 外部访问我的IP限制，最大跨度不超过32，目前是1~24它们是 CIDR 表示法。
            - My=gold
#            - name: My
#              args:
#                userType: diamond

        - id: pay_routh2 #pay_routh2                #路由的ID(类似mysql主键ID)，没有固定规则但要求唯一，建议配合服务名
          #uri: http://localhost:8001                #匹配后提供服务的路由地址
          uri: lb://cloud-payment-service
          predicates:
            - Path=/pay/gateway/info/**              # 断言，路径相匹配的进行路由

        - id: pay_routh3 #pay_routh3
          uri: lb://cloud-payment-service                #匹配后提供服务的路由地址
          predicates:
            - Path=/pay/gateway/filter/**              # 断言，路径相匹配的进行路由
          filters:
            - AddRequestHeader=X-Request-atguigu1,atguiguValue1  # 请求头kv，若一头含有多参则重写一行设置
            - AddRequestHeader=X-Request-atguigu2,atguiguValue2
            - RemoveRequestHeader=sec-fetch-site      # 删除请求头sec-fetch-site
```

**修改后**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887609720-03219139-b551-4ffb-bab7-8fd69c5450ff.png)

**6.29. The SetRequestHeader GatewayFilter Factory 修改请求头ByName**

**修改前(sec-fetch-mode)**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887609817-671c270f-3637-446d-8589-a754097bab50.png)

**YML**

```plain
server:
  port: 9527

spring:
  application:
    name: cloud-gateway #以微服务注册进consul或nacos服务列表内
  cloud:
    consul: #配置consul地址
      host: localhost
      port: 8500
      discovery:
        prefer-ip-address: true
        service-name: ${spring.application.name}
    gateway:
      routes:
        - id: pay_routh1 #pay_routh1                #路由的ID(类似mysql主键ID)，没有固定规则但要求唯一，建议配合服务名
          #uri: http://localhost:8001                #匹配后提供服务的路由地址
          uri: lb://cloud-payment-service                #匹配后提供服务的路由地址
          predicates:
            - Path=/pay/gateway/get/**              # 断言，路径相匹配的进行路由
            - After=2023-12-30T23:02:39.079979400+08:00[Asia/Shanghai]
            #- Cookie=username,zzyy
            # - Header=X-Request-Id, \d+ # 请求头要有X-Request-Id属性并且值为整数的正则表达式
            #- Host=**.atguigu.com
            #- Query=username, \d+  # 要有参数名username并且值还要是整数才能路由
            #- RemoteAddr=192.168.124.1/24 # 外部访问我的IP限制，最大跨度不超过32，目前是1~24它们是 CIDR 表示法。
            - My=gold
#            - name: My
#              args:
#                userType: diamond

        - id: pay_routh2 #pay_routh2                #路由的ID(类似mysql主键ID)，没有固定规则但要求唯一，建议配合服务名
          #uri: http://localhost:8001                #匹配后提供服务的路由地址
          uri: lb://cloud-payment-service
          predicates:
            - Path=/pay/gateway/info/**              # 断言，路径相匹配的进行路由

        - id: pay_routh3 #pay_routh3
          uri: lb://cloud-payment-service                #匹配后提供服务的路由地址
          predicates:
            - Path=/pay/gateway/filter/**              # 断言，路径相匹配的进行路由
          filters:
            - AddRequestHeader=X-Request-atguigu1,atguiguValue1  # 请求头kv，若一头含有多参则重写一行设置
            - AddRequestHeader=X-Request-atguigu2,atguiguValue2
            - RemoveRequestHeader=sec-fetch-site      # 删除请求头sec-fetch-site
            - SetRequestHeader=sec-fetch-mode, Blue-updatebyzzyy # 将请求头sec-fetch-mode对应的值修改为Blue-updatebyzzyy
```



**修改后**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887617182-17069dd5-f1b3-4def-8e34-875050312c4b.png)

###### 2 请求参数(RequestParameter)相关组
**6.3. The AddRequestParameter GatewayFilter Factory**

**6.19. The RemoveRequestParameter GatewayFilter Factory**

**上述两个合一块**

**YML**

```plain
server:
  port: 9527

spring:
  application:
    name: cloud-gateway #以微服务注册进consul或nacos服务列表内
  cloud:
    consul: #配置consul地址
      host: localhost
      port: 8500
      discovery:
        prefer-ip-address: true
        service-name: ${spring.application.name}
    gateway:
      routes:
        - id: pay_routh1 #pay_routh1                #路由的ID(类似mysql主键ID)，没有固定规则但要求唯一，建议配合服务名
          #uri: http://localhost:8001                #匹配后提供服务的路由地址
          uri: lb://cloud-payment-service                #匹配后提供服务的路由地址
          predicates:
            - Path=/pay/gateway/get/**              # 断言，路径相匹配的进行路由
            - After=2023-12-30T23:02:39.079979400+08:00[Asia/Shanghai]
            #- Cookie=username,zzyy
            # - Header=X-Request-Id, \d+ # 请求头要有X-Request-Id属性并且值为整数的正则表达式
            #- Host=**.atguigu.com
            #- Query=username, \d+  # 要有参数名username并且值还要是整数才能路由
            #- RemoteAddr=192.168.124.1/24 # 外部访问我的IP限制，最大跨度不超过32，目前是1~24它们是 CIDR 表示法。
            - My=gold
#            - name: My
#              args:
#                userType: diamond

        - id: pay_routh2 #pay_routh2                #路由的ID(类似mysql主键ID)，没有固定规则但要求唯一，建议配合服务名
          #uri: http://localhost:8001                #匹配后提供服务的路由地址
          uri: lb://cloud-payment-service
          predicates:
            - Path=/pay/gateway/info/**              # 断言，路径相匹配的进行路由

        - id: pay_routh3 #pay_routh3
          uri: lb://cloud-payment-service                #匹配后提供服务的路由地址
          predicates:
            - Path=/pay/gateway/filter/**              # 断言，路径相匹配的进行路由
          filters:
            - AddRequestHeader=X-Request-atguigu1,atguiguValue1  # 请求头kv，若一头含有多参则重写一行设置
            - AddRequestHeader=X-Request-atguigu2,atguiguValue2
            - RemoveRequestHeader=sec-fetch-site      # 删除请求头sec-fetch-site
            - SetRequestHeader=sec-fetch-mode, Blue-updatebyzzyy # 将请求头sec-fetch-mode对应的值修改为Blue-updatebyzzyy
            - AddRequestParameter=customerId,9527001 # 新增请求参数Parameter：k ，v
            - RemoveRequestParameter=customerName   # 删除url请求参数customerName，你传递过来也是null
```

**修改PayGateWayController**

```plain
@GetMapping(value = "/pay/gateway/filter")
public ResultData<String> getGatewayFilter(HttpServletRequest request)
{
    String result = "";
    Enumeration<String> headers = request.getHeaderNames();
    while(headers.hasMoreElements())
    {
        String headName = headers.nextElement();
        String headValue = request.getHeader(headName);
        System.out.println("request headName:" + headName +"---"+"request headValue:" + headValue);

        if(headName.equalsIgnoreCase("X-Request-atguigu1")
                || headName.equalsIgnoreCase("X-Request-atguigu2")) {
            result = result+headName + "\t " + headValue +" ";
        }
    }

    System.out.println("=============================================");
    String customerId = request.getParameter("customerId");
    System.out.println("request Parameter customerId: "+customerId);

    String customerName = request.getParameter("customerName");
    System.out.println("request Parameter customerName: "+customerName);
    System.out.println("=============================================");

    return ResultData.success("getGatewayFilter 过滤器 test： "+result+" \t "+ DateUtil.now());
}
```

**测试**

[http://localhost:9527/pay/gateway/filter](http://localhost:9527/pay/gateway/filter)

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887616527-13a999cd-c1e0-46dc-b53b-7f41e42b6082.png)

[http://localhost:9527/pay/gateway/filter?customerId=9999&customerName=z3](http://localhost:9527/pay/gateway/filter?customerId=9999&customerName=z3)

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887618950-80527964-5974-4c48-aa3d-5ca26baa5d27.png)

**3 回应头(ResponseHeader)相关组**

**开启配置前，按照地址chrome查看一下**

[http://localhost:9527/pay/gateway/filter](http://localhost:9527/pay/gateway/filter)

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887618850-c8b6aac1-1627-45c2-a9f8-94f49b4b2a18.png)

**6.4. The AddResponseHeader GatewayFilter Factory**

YML

```plain
server:
  port: 9527

spring:
  application:
    name: cloud-gateway #以微服务注册进consul或nacos服务列表内
  cloud:
    consul: #配置consul地址
      host: localhost
      port: 8500
      discovery:
        prefer-ip-address: true
        service-name: ${spring.application.name}
    gateway:
      routes:
        - id: pay_routh1 #pay_routh1                #路由的ID(类似mysql主键ID)，没有固定规则但要求唯一，建议配合服务名
          #uri: http://localhost:8001                #匹配后提供服务的路由地址
          uri: lb://cloud-payment-service                #匹配后提供服务的路由地址
          predicates:
            - Path=/pay/gateway/get/**              # 断言，路径相匹配的进行路由
            - After=2023-12-30T23:02:39.079979400+08:00[Asia/Shanghai]
            #- Cookie=username,zzyy
            # - Header=X-Request-Id, \d+ # 请求头要有X-Request-Id属性并且值为整数的正则表达式
            #- Host=**.atguigu.com
            #- Query=username, \d+  # 要有参数名username并且值还要是整数才能路由
            #- RemoteAddr=192.168.124.1/24 # 外部访问我的IP限制，最大跨度不超过32，目前是1~24它们是 CIDR 表示法。
            - My=gold
#            - name: My
#              args:
#                userType: diamond

        - id: pay_routh2 #pay_routh2                #路由的ID(类似mysql主键ID)，没有固定规则但要求唯一，建议配合服务名
          #uri: http://localhost:8001                #匹配后提供服务的路由地址
          uri: lb://cloud-payment-service
          predicates:
            - Path=/pay/gateway/info/**              # 断言，路径相匹配的进行路由

        - id: pay_routh3 #pay_routh3
          uri: lb://cloud-payment-service                #匹配后提供服务的路由地址
          predicates:
            - Path=/pay/gateway/filter/**              # 断言，路径相匹配的进行路由
          filters:
            - AddRequestHeader=X-Request-atguigu1,atguiguValue1  # 请求头kv，若一头含有多参则重写一行设置
            - AddRequestHeader=X-Request-atguigu2,atguiguValue2
            - RemoveRequestHeader=sec-fetch-site      # 删除请求头sec-fetch-site
            - SetRequestHeader=sec-fetch-mode, Blue-updatebyzzyy # 将请求头sec-fetch-mode对应的值修改为Blue-updatebyzzyy
            - AddRequestParameter=customerId,9527001 # 新增请求参数Parameter：k ，v
            - RemoveRequestParameter=customerName   # 删除url请求参数customerName，你传递过来也是null
            - AddResponseHeader=X-Response-atguigu, BlueResponse # 新增请求参数X-Response-atguigu并设值为BlueResponse
```

**6.30. The SetResponseHeader GatewayFilter Factory**

YML

```plain
server:
  port: 9527

spring:
  application:
    name: cloud-gateway #以微服务注册进consul或nacos服务列表内
  cloud:
    consul: #配置consul地址
      host: localhost
      port: 8500
      discovery:
        prefer-ip-address: true
        service-name: ${spring.application.name}
    gateway:
      routes:
        - id: pay_routh1 #pay_routh1                #路由的ID(类似mysql主键ID)，没有固定规则但要求唯一，建议配合服务名
          #uri: http://localhost:8001                #匹配后提供服务的路由地址
          uri: lb://cloud-payment-service                #匹配后提供服务的路由地址
          predicates:
            - Path=/pay/gateway/get/**              # 断言，路径相匹配的进行路由
            - After=2023-12-30T23:02:39.079979400+08:00[Asia/Shanghai]
            #- Cookie=username,zzyy
            # - Header=X-Request-Id, \d+ # 请求头要有X-Request-Id属性并且值为整数的正则表达式
            #- Host=**.atguigu.com
            #- Query=username, \d+  # 要有参数名username并且值还要是整数才能路由
            #- RemoteAddr=192.168.124.1/24 # 外部访问我的IP限制，最大跨度不超过32，目前是1~24它们是 CIDR 表示法。
            - My=gold
#            - name: My
#              args:
#                userType: diamond

        - id: pay_routh2 #pay_routh2                #路由的ID(类似mysql主键ID)，没有固定规则但要求唯一，建议配合服务名
          #uri: http://localhost:8001                #匹配后提供服务的路由地址
          uri: lb://cloud-payment-service
          predicates:
            - Path=/pay/gateway/info/**              # 断言，路径相匹配的进行路由

        - id: pay_routh3 #pay_routh3
          uri: lb://cloud-payment-service                #匹配后提供服务的路由地址
          predicates:
            - Path=/pay/gateway/filter/**              # 断言，路径相匹配的进行路由
          filters:
            - AddRequestHeader=X-Request-atguigu1,atguiguValue1  # 请求头kv，若一头含有多参则重写一行设置
            - AddRequestHeader=X-Request-atguigu2,atguiguValue2
            - RemoveRequestHeader=sec-fetch-site      # 删除请求头sec-fetch-site
            - SetRequestHeader=sec-fetch-mode, Blue-updatebyzzyy # 将请求头sec-fetch-mode对应的值修改为Blue-updatebyzzyy
            - AddRequestParameter=customerId,9527001 # 新增请求参数Parameter：k ，v
            - RemoveRequestParameter=customerName   # 删除url请求参数customerName，你传递过来也是null
            - AddResponseHeader=X-Response-atguigu, BlueResponse # 新增请求参数X-Response-atguigu并设值为BlueResponse
            - SetResponseHeader=Date,2099-11-11 # 设置回应头Date值为2099-11-11
```

**6.20. The RemoveResponseHeader GatewayFilter Factory**

YML

```plain
server:
  port: 9527

spring:
  application:
    name: cloud-gateway #以微服务注册进consul或nacos服务列表内
  cloud:
    consul: #配置consul地址
      host: localhost
      port: 8500
      discovery:
        prefer-ip-address: true
        service-name: ${spring.application.name}
    gateway:
      routes:
        - id: pay_routh1 #pay_routh1                #路由的ID(类似mysql主键ID)，没有固定规则但要求唯一，建议配合服务名
          #uri: http://localhost:8001                #匹配后提供服务的路由地址
          uri: lb://cloud-payment-service                #匹配后提供服务的路由地址
          predicates:
            - Path=/pay/gateway/get/**              # 断言，路径相匹配的进行路由
            - After=2023-12-30T23:02:39.079979400+08:00[Asia/Shanghai]
            #- Cookie=username,zzyy
            # - Header=X-Request-Id, \d+ # 请求头要有X-Request-Id属性并且值为整数的正则表达式
            #- Host=**.atguigu.com
            #- Query=username, \d+  # 要有参数名username并且值还要是整数才能路由
            #- RemoteAddr=192.168.124.1/24 # 外部访问我的IP限制，最大跨度不超过32，目前是1~24它们是 CIDR 表示法。
            - My=gold
#            - name: My
#              args:
#                userType: diamond

        - id: pay_routh2 #pay_routh2                #路由的ID(类似mysql主键ID)，没有固定规则但要求唯一，建议配合服务名
          #uri: http://localhost:8001                #匹配后提供服务的路由地址
          uri: lb://cloud-payment-service
          predicates:
            - Path=/pay/gateway/info/**              # 断言，路径相匹配的进行路由

        - id: pay_routh3 #pay_routh3
          uri: lb://cloud-payment-service                #匹配后提供服务的路由地址
          predicates:
            - Path=/pay/gateway/filter/**              # 断言，路径相匹配的进行路由
          filters:
            - AddRequestHeader=X-Request-atguigu1,atguiguValue1  # 请求头kv，若一头含有多参则重写一行设置
            - AddRequestHeader=X-Request-atguigu2,atguiguValue2
            - RemoveRequestHeader=sec-fetch-site      # 删除请求头sec-fetch-site
            - SetRequestHeader=sec-fetch-mode, Blue-updatebyzzyy # 将请求头sec-fetch-mode对应的值修改为Blue-updatebyzzyy
            - AddRequestParameter=customerId,9527001 # 新增请求参数Parameter：k ，v
            - RemoveRequestParameter=customerName   # 删除url请求参数customerName，你传递过来也是null
            - AddResponseHeader=X-Response-atguigu, BlueResponse # 新增请求参数X-Response-atguigu并设值为BlueResponse
            - SetResponseHeader=Date,2099-11-11 # 设置回应头Date值为2099-11-11
            - RemoveResponseHeader=Content-Type # 将默认自带Content-Type回应属性删除
```

**开启配置后，上面三个配置打包一块上**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887618313-9af2e2c3-8f81-4285-a401-7a779ea4389d.png)

**4 前缀和路径相关组**

**6.14. The PrefixPath GatewayFilter Factory 自动添加路径前缀**

**之前的正确地址**

[http://localhost:9527/pay/gateway/filter](http://localhost:9527/pay/gateway/filter)

**YML**

```plain
server:
  port: 9527

spring:
  application:
    name: cloud-gateway #以微服务注册进consul或nacos服务列表内
  cloud:
    consul: #配置consul地址
      host: localhost
      port: 8500
      discovery:
        prefer-ip-address: true
        service-name: ${spring.application.name}
    gateway:
      routes:
        - id: pay_routh1 #pay_routh1                #路由的ID(类似mysql主键ID)，没有固定规则但要求唯一，建议配合服务名
          #uri: http://localhost:8001                #匹配后提供服务的路由地址
          uri: lb://cloud-payment-service          #匹配后提供服务的路由地址
          predicates:
            - Path=/pay/gateway/get/**              # 断言，路径相匹配的进行路由
            #- After=2023-11-20T17:38:13.586918800+08:00[Asia/Shanghai]
            - Before=2023-12-29T17:58:13.586918800+08:00[Asia/Shanghai]
            #- Between=2023-11-21T17:38:13.586918800+08:00[Asia/Shanghai],2023-11-22T17:38:13.586918800+08:00[Asia/Shanghai]
            #- Cookie=username,zzyy
            #- Header=X-Request-Id, \d+  # 请求头要有X-Request-Id属性并且值为整数的正则表达式
            #- Host=**.atguigu.com
            #- Query=username, \d+  # 要有参数名username并且值还要是整数才能路由
            #- RemoteAddr=192.168.124.1/24 # 外部访问我的IP限制，最大跨度不超过32，目前是1~24。

        - id: pay_routh2 #pay_routh2                #路由的ID(类似mysql主键ID)，没有固定规则但要求唯一，建议配合服务名
          #uri: http://localhost:8001                #匹配后提供服务的路由地址
          uri: lb://cloud-payment-service                #匹配后提供服务的路由地址
          predicates:
            - Path=/pay/gateway/info/**              # 断言，路径相匹配的进行路由


        - id: pay_routh3 #pay_routh3
          uri: lb://cloud-payment-service                #匹配后提供服务的路由地址
          predicates:
            #- Path=/pay/gateway/filter/**   # 被分拆为: PrefixPath + Path

            - Path=/gateway/filter/**              # 断言，为配合PrefixPath测试过滤，暂时注释掉/pay
          filters:
            - AddRequestHeader=X-Request-atguigu1,atguiguValue1  #请求头kv，若一头含有多参则重写一行设置
            #- AddRequestHeader=X-Request-atguigu2,atguiguValue2
            #- RemoveRequestHeader=sec-fetch-site      # 删除请求头sec-fetch-site
            #- SetRequestHeader=sec-fetch-mode, Blue-updatebyzzyy # 将请求头sec-fetch-mode对应的值修改为Blue-updatebyzzyy
            #- AddRequestParameter=customerId,9527001 # 新增请求参数Parameter：k ，v
            #- RemoveRequestParameter=customerName   # 删除url请求参数customerName，你传递过来也是null
            #- AddResponseHeader=X-Response-atguigu, BlueResponse # 新增请求参数X-Response-atguigu并设值为BlueResponse
            #- SetResponseHeader=Date,2099-11-11 # 设置回应头Date值为2099-11-11
            #- RemoveResponseHeader=Content-Type # 将默认自带Content-Type回应属性删除
            - PrefixPath=/pay # http://localhost:9527/pay/gateway/filter
```

分拆说明 

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887621509-d79fa223-3ccb-4b24-bb25-68ec123e89d1.png)

| 之前完整正确地址： | http://localhost:9527/pay/gateway/filter |
| :--- | :--- |
| 现在完整组合地址： | PrefixPath + Path |
| 实际调用地址： | http://localhost:9527/gateway/filter<br/>**相当于说前缀被过滤器统一管理了。** |


**Chrome测试**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887623318-7eb1420c-3e24-4c7d-bd26-511b097ab80c.png)

**6.29. The SetPath GatewayFilter Factory**

**访问路径修改**

**测试**

**YML**

```plain
server:
  port: 9527

spring:
  application:
    name: cloud-gateway #以微服务注册进consul或nacos服务列表内
  cloud:
    consul: #配置consul地址
      host: localhost
      port: 8500
      discovery:
        prefer-ip-address: true
        service-name: ${spring.application.name}
    gateway:
      routes:
        - id: pay_routh1 #pay_routh1                #路由的ID(类似mysql主键ID)，没有固定规则但要求唯一，建议配合服务名
          #uri: http://localhost:8001                #匹配后提供服务的路由地址
          uri: lb://cloud-payment-service          #匹配后提供服务的路由地址
          predicates:
            - Path=/pay/gateway/get/**              # 断言，路径相匹配的进行路由
            #- After=2023-11-20T17:38:13.586918800+08:00[Asia/Shanghai]
            - Before=2023-12-29T17:58:13.586918800+08:00[Asia/Shanghai]
            #- Between=2023-11-21T17:38:13.586918800+08:00[Asia/Shanghai],2023-11-22T17:38:13.586918800+08:00[Asia/Shanghai]
            #- Cookie=username,zzyy
            #- Header=X-Request-Id, \d+  # 请求头要有X-Request-Id属性并且值为整数的正则表达式
            #- Host=**.atguigu.com
            #- Query=username, \d+  # 要有参数名username并且值还要是整数才能路由
            #- RemoteAddr=192.168.1.196/24 # 外部访问我的IP限制，最大跨度不超过32，目前是1~24。

        - id: pay_routh2 #pay_routh2                #路由的ID(类似mysql主键ID)，没有固定规则但要求唯一，建议配合服务名
          #uri: http://localhost:8001                #匹配后提供服务的路由地址
          uri: lb://cloud-payment-service                #匹配后提供服务的路由地址
          predicates:
            - Path=/pay/gateway/info/**              # 断言，路径相匹配的进行路由


        - id: pay_routh3 #pay_routh3
          uri: lb://cloud-payment-service                #匹配后提供服务的路由地址
          predicates:
            #- Path=/pay/gateway/filter/** # 真实地址
            #- Path=/gateway/filter/**              # 断言，为配合PrefixPath测试过滤，暂时注释掉/pay
            - Path=/XYZ/abc/{segment}           # 断言，为配合SetPath测试，{segment}的内容最后被SetPath取代
          filters:
            - AddRequestHeader=X-Request-atguigu1,atguiguValue1  #请求头kv，若一头含有多参则重写一行设置
                        #- AddRequestHeader=X-Request-atguigu2,atguiguValue2
            #- RemoveRequestHeader=sec-fetch-site      # 删除请求头sec-fetch-site
            #- SetRequestHeader=sec-fetch-mode, Blue-updatebyzzyy # 将请求头sec-fetch-mode对应的值修改为Blue-updatebyzzyy
            #- AddRequestParameter=customerId,9527001 # 新增请求参数Parameter：k ，v
            #- RemoveRequestParameter=customerName   # 删除url请求参数customerName，你传递过来也是null
            #- AddResponseHeader=X-Response-atguigu, BlueResponse # 新增请求参数X-Response-atguigu并设值为BlueResponse
            #- SetResponseHeader=Date,2099-11-11 # 设置回应头Date值为2099-11-11
            #- RemoveResponseHeader=Content-Type # 将默认自带Content-Type回应属性删除
                        #- PrefixPath=/pay # http://localhost:9527/pay/gateway/filter
            - SetPath=/pay/gateway/{segment}  # {segment}表示占位符，你写abc也行但要上下一致
```

说明：

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887625433-78afa554-cd71-46a4-b6f6-9cf557a5ecad.png)

| /XYZ/abc/{segment} | {segment}就是个占位符，等价于SetPath后面指定的{segment}内容 |  |
| :--- | :--- | --- |


浏览器访问地址: http://localhost:9527/XYZ/abc/filter

实际微服务地址：http://localhost:9527/pay/gateway/filter

[http://localhost:9527/XYZ/abc/filter](http://localhost:9527/XYZ/abc/filter)

**结果**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887625410-c43f49ee-56fe-4884-926a-86a33e63cb78.png)

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887627025-4fa58e51-9c26-44d7-b8a3-5ca3197e26ee.png)

**6.16. The RedirectTo GatewayFilter Factory**

重定向到某个页面

[http://localhost:9527/pay/gateway/filter](http://localhost:9527/pay/gateway/filter)

YML

```plain
server:
  port: 9527

spring:
  application:
    name: cloud-gateway #以微服务注册进consul或nacos服务列表内
  cloud:
    consul: #配置consul地址
      host: localhost
      port: 8500
      discovery:
        prefer-ip-address: true
        service-name: ${spring.application.name}
    gateway:
      routes:
        - id: pay_routh1 #pay_routh1                #路由的ID(类似mysql主键ID)，没有固定规则但要求唯一，建议配合服务名
          #uri: http://localhost:8001                #匹配后提供服务的路由地址
          uri: lb://cloud-payment-service          #匹配后提供服务的路由地址
          predicates:
            - Path=/pay/gateway/get/**              # 断言，路径相匹配的进行路由
            #- After=2023-11-20T17:38:13.586918800+08:00[Asia/Shanghai]
            - Before=2023-12-29T17:58:13.586918800+08:00[Asia/Shanghai]
            #- Between=2023-11-21T17:38:13.586918800+08:00[Asia/Shanghai],2023-11-22T17:38:13.586918800+08:00[Asia/Shanghai]
            #- Cookie=username,zzyy
            #- Header=X-Request-Id, \d+  # 请求头要有X-Request-Id属性并且值为整数的正则表达式
            #- Host=**.atguigu.com
            #- Query=username, \d+  # 要有参数名username并且值还要是整数才能路由
            #- RemoteAddr=192.168.1.196/24 # 外部访问我的IP限制，最大跨度不超过32，目前是1~24。

        - id: pay_routh2 #pay_routh2                #路由的ID(类似mysql主键ID)，没有固定规则但要求唯一，建议配合服务名
          #uri: http://localhost:8001                #匹配后提供服务的路由地址
          uri: lb://cloud-payment-service                #匹配后提供服务的路由地址
          predicates:
            - Path=/pay/gateway/info/**              # 断言，路径相匹配的进行路由


        - id: pay_routh3 #pay_routh3
          uri: lb://cloud-payment-service                #匹配后提供服务的路由地址
          predicates:
            - Path=/pay/gateway/filter/** # 真实地址
            #- Path=/gateway/filter/**              # 断言，为配合PrefixPath测试过滤，暂时注释掉/pay
            #- Path=/XYZ/abc/{segment}           # 断言，为配合SetPath测试，{segment}的内容最后被SetPath取代
          filters:
            - AddRequestHeader=X-Request-atguigu1,atguiguValue1  #请求头kv，若一头含有多参则重写一行设置
            #- AddRequestHeader=X-Request-atguigu2,atguiguValue2
            #- RemoveRequestHeader=sec-fetch-site      # 删除请求头sec-fetch-site
            #- SetRequestHeader=sec-fetch-mode, Blue-updatebyzzyy # 将请求头sec-fetch-mode对应的值修改为Blue-updatebyzzyy
            #- AddRequestParameter=customerId,9527001 # 新增请求参数Parameter：k ，v
            #- RemoveRequestParameter=customerName   # 删除url请求参数customerName，你传递过来也是null
            #- AddResponseHeader=X-Response-atguigu, BlueResponse # 新增请求参数X-Response-atguigu并设值为BlueResponse
            #- SetResponseHeader=Date,2099-11-11 # 设置回应头Date值为2099-11-11
            #- RemoveResponseHeader=Content-Type # 将默认自带Content-Type回应属性删除
            #- PrefixPath=/pay # http://localhost:9527/pay/gateway/filter
            - RedirectTo=302, http://www.atguigu.com/ # 访问http://localhost:9527/pay/gateway/filter跳转到http://www.atguigu.com/
```

**5 其它**

**6.38. Default Filters**

**配置在此处相当于全局通用，自定义秒变Global**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887629245-b844aa4e-c35b-4a44-b0e3-60a0425526b5.png)

**本次案例全部YML配置全集**

```plain
server:
  port: 9527

spring:
  application:
    name: cloud-gateway #以微服务注册进consul或nacos服务列表内
  cloud:
    consul: #配置consul地址
      host: localhost
      port: 8500
      discovery:
        prefer-ip-address: true
        service-name: ${spring.application.name}
    gateway:
      routes:
        - id: pay_routh1 #pay_routh1                #路由的ID(类似mysql主键ID)，没有固定规则但要求唯一，建议配合服务名
          #uri: http://localhost:8001                #匹配后提供服务的路由地址
          uri: lb://cloud-payment-service                #匹配后提供服务的路由地址
          predicates:
            - Path=/pay/gateway/get/**              # 断言，路径相匹配的进行路由
            - After=2023-12-30T23:02:39.079979400+08:00[Asia/Shanghai]
            #- Cookie=username,zzyy
            # - Header=X-Request-Id, \d+ # 请求头要有X-Request-Id属性并且值为整数的正则表达式
            #- Host=**.atguigu.com
            #- Query=username, \d+  # 要有参数名username并且值还要是整数才能路由
            #- RemoteAddr=192.168.124.1/24 # 外部访问我的IP限制，最大跨度不超过32，目前是1~24它们是 CIDR 表示法。
            - My=gold
#            - name: My
#              args:
#                userType: diamond

        - id: pay_routh2 #pay_routh2                #路由的ID(类似mysql主键ID)，没有固定规则但要求唯一，建议配合服务名
          #uri: http://localhost:8001                #匹配后提供服务的路由地址
          uri: lb://cloud-payment-service
          predicates:
            - Path=/pay/gateway/info/**              # 断言，路径相匹配的进行路由

        - id: pay_routh3 #pay_routh3
          uri: lb://cloud-payment-service                #匹配后提供服务的路由地址
          predicates:
            - Path=/pay/gateway/filter/**              # 断言，路径相匹配的进行路由，默认正确地址
            #- Path=/gateway/filter/**              # 断言，为配合PrefixPath测试过滤，暂时注释掉/pay
            #- Path=/XYZ/abc/{segment}           # 断言，为配合SetPath测试，{segment}的内容最后被SetPath取代
          filters:
            - RedirectTo=302, http://www.atguigu.com/ # 访问http://localhost:9527/pay/gateway/filter跳转到http://www.atguigu.com/
            #- SetPath=/pay/gateway/{segment}  # {segment}表示占位符，你写abc也行但要上下一致
            #- PrefixPath=/pay # http://localhost:9527/pay/gateway/filter  被分拆为: PrefixPath + Path
            #- AddRequestHeader=X-Request-atguigu1,atguiguValue1  # 请求头kv，若一头含有多参则重写一行设置
            #- AddRequestHeader=X-Request-atguigu2,atguiguValue2
            #- RemoveRequestHeader=sec-fetch-site      # 删除请求头sec-fetch-site
            #- SetRequestHeader=sec-fetch-mode, Blue-updatebyzzyy # 将请求头sec-fetch-mode对应的值修改为Blue-updatebyzzyy
            #- AddRequestParameter=customerId,9527001 # 新增请求参数Parameter：k ，v
            #- RemoveRequestParameter=customerName   # 删除url请求参数customerName，你传递过来也是null
            #- AddResponseHeader=X-Response-atguigu, BlueResponse # 新增请求参数X-Response-atguigu并设值为BlueResponse
            #- SetResponseHeader=Date,2099-11-11 # 设置回应头Date值为2099-11-11
            #- RemoveResponseHeader=Content-Type # 将默认自带Content-Type回应属性删除
```

##### **9.6.3.3 **Gateway自定义过滤器
**1 自定义全局Filter**

**1.1 面试题**

统计接口调用耗时情况，如何落地，谈谈设计思路

通过自定义全局过滤器搞定上述需求

**1.2 案例**

**自定义接口调用耗时统计的全局过滤器**

**步骤**

**新建类MyGlobalFilter并实现GlobalFilter,Ordered两个接口**

```java
@Component //不要忘记
@Slf4j
public class MyGlobalFilter implements GlobalFilter, Ordered{
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain){
        return null;
    }

    @Override
    public int getOrder(){
        return 0;
    }
}
```

**YML**

```yaml
server:
  port: 9527

spring:
  application:
    name: cloud-gateway #以微服务注册进consul或nacos服务列表内
  cloud:
    consul: #配置consul地址
      host: localhost
      port: 8500
      discovery:
        prefer-ip-address: true
        service-name: ${spring.application.name}
    gateway:
      routes:
        - id: pay_routh1 #pay_routh1                #路由的ID(类似mysql主键ID)，没有固定规则但要求唯一，建议配合服务名
          uri: lb://cloud-payment-service                #匹配后提供服务的路由地址
          predicates:
            - Path=/pay/gateway/get/**              # 断言，路径相匹配的进行路由
            - After=2023-12-30T23:02:39.079979400+08:00[Asia/Shanghai]

        - id: pay_routh2 #pay_routh2                #路由的ID(类似mysql主键ID)，没有固定规则但要求唯一，建议配合服务名
          uri: lb://cloud-payment-service
          predicates:
            - Path=/pay/gateway/info/**              # 断言，路径相匹配的进行路由

        - id: pay_routh3 #pay_routh3
          uri: lb://cloud-payment-service                #匹配后提供服务的路由地址
          predicates:
            - Path=/pay/gateway/filter/**              # 断言，路径相匹配的进行路由，默认正确地址
          filters:
            - AddRequestHeader=X-Request-atguigu1,atguiguValue1  # 请求头kv，若一头含有多参则重写一行设置
```

**code**

```java
@Slf4j // Lombok注解，自动生成log日志对象
@Component // Spring注解，将该类声明为Spring容器管理的组件
public class MyGlobalFilter implements GlobalFilter, Ordered {

    /**
     * 开始访问时间常量
     * 用于在exchange属性中存储请求开始时间的时间戳
     */
    private static final String BEGIN_VISIT_TIME = "begin_visit_time";

    /**
     * 获取过滤器执行顺序
     * 实现Ordered接口的方法，定义过滤器在链中的执行顺序
     * 
     * @return int 返回过滤器的执行顺序，数值越小优先级越高
     */
    @Override
    public int getOrder() {
        return 0; // 返回0表示最高优先级
    }

    /**
     * 过滤器核心处理方法
     * 实现GlobalFilter接口的方法，处理所有经过网关的请求
     *
     * @param exchange 服务器Web交换对象，包含HTTP请求和响应的所有信息
     * @param chain 网关过滤器链，用于继续执行后续的过滤器
     * @return Mono<Void> 响应式编程的返回类型，表示异步空结果
     */
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        // 在请求属性中记录当前时间作为开始时间戳
        exchange.getAttributes().put(BEGIN_VISIT_TIME, System.currentTimeMillis());
        // 执行过滤器链并返回结果，然后在完成后执行统计日志输出
        return chain.filter(exchange).then(Mono.fromRunnable(() -> {
            // 从exchange属性中获取请求开始时间
            Long beginVisitTime = exchange.getAttribute(BEGIN_VISIT_TIME);
            // 检查开始时间是否存在，避免空指针异常
            if (beginVisitTime != null) {
                // 记录访问的主机信息
                log.info("访问接口主机: " + exchange.getRequest().getURI().getHost());
                // 记录访问的端口信息
                log.info("访问接口端口: " + exchange.getRequest().getURI().getPort());
                // 记录访问的URL路径信息
                log.info("访问接口URL: " + exchange.getRequest().getURI().getPath());
                // 记录URL查询参数信息
                log.info("访问接口URL参数: " + exchange.getRequest().getURI().getRawQuery());
                // 计算并记录请求处理总时长（毫秒）
                log.info("访问接口时长: " + (System.currentTimeMillis() - beginVisitTime) + "ms");
                // 输出分隔线，便于日志阅读
                log.info("我是美丽分割线: ###################################################");
                // 输出空行，进一步增强日志可读性
                System.out.println();
            }
        }));
    }
}
```

**测试**

[http://localhost:9527/pay/gateway/info](http://localhost:9527/pay/gateway/info)

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887659862-fd4ab794-21ae-4d50-9103-05259bfe2bc4.png)

[http://localhost:9527/pay/gateway/get/1](http://localhost:9527/pay/gateway/get/1)

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887659854-25094ba3-f8ca-4cbc-a1ea-93873abbd970.png)

[http://localhost:9527/pay/gateway/filter](http://localhost:9527/pay/gateway/filter)

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887659881-4a2b66d2-4142-4852-9a70-319ae3a4d900.png)

**2 自定义条件Filter**

**自定义，单一内置过滤器GatewayFilter**

**先参考GateWay内置出厂默认的**

+ **SetStatusGatewayFilterFactory**
+ **SetPathGatewayFilterFactory**
+ **AddResponseHeaderGatewayFilterFactory**

**自定义网关过滤器规则步骤套路** 

1. **新建类名XXX需要以GatewayFilterFactory结尾并继承AbstractGatewayFilterFactory类**

```java
@Component//标注不可忘
public class MyGatewayFilterFactory extends AbstractGatewayFilterFactory<MyGatewayFilterFactory.Config>{

}
```

2. **新建XXXGatewayFilterFactory.Config内部类**

```java
@Component
public class MyGatewayFilterFactory extends AbstractGatewayFilterFactory<MyGatewayFilterFactory.Config>{
    public static class Config {
        //设定一个状态值/标志位，它等于多少，匹配和才可以访问
        @Setter @Getter
        private String status;
    }
}
```

**完整代码01**

```java
@Component
public class MyGatewayFilterFactory extends AbstractGatewayFilterFactory<MyGatewayFilterFactory.Config>{
    public MyGatewayFilterFactory(){
        super(MyGatewayFilterFactory.Config.class);
    }

    @Override
    public GatewayFilter apply(MyGatewayFilterFactory.Config config){
        return new GatewayFilter(){
            @Override
            public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain){
                ServerHttpRequest request = exchange.getRequest();
                System.out.println("进入了自定义网关过滤器MyGatewayFilterFactory，status："+config.getStatus());
                if(request.getQueryParams().containsKey("atguigu")){
                    return chain.filter(exchange);
                }else{
                    exchange.getResponse().setStatusCode(HttpStatus.BAD_REQUEST);
                    return exchange.getResponse().setComplete();
                }
            }
        };
    }

    @Override
    public List<String> shortcutFieldOrder(){
        return Arrays.asList("status");
    }

    public static class Config{
        //设定一个状态值/标志位，它等于多少，匹配和才可以访问
        @Getter@Setter
        private String status;
    }
}
//单一内置过滤器GatewayFilter
```

**YML**

My补充说明：

**出厂默认**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887659860-f5ea468d-809a-43de-9fce-f4ca6b5b34d9.png)

**自己定制My**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887660203-574019d5-e1fa-42cc-baa4-c3e41a292afd.png)

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887667236-4ffd7ca2-b865-4a57-b30f-a0a0cb233dab.png)

**测试**

✖  

✔ ?atguigu=java

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1756887667127-83f7a448-9792-4a9a-ae2d-c67fa5ca5f78.png)

