## elasticsearch(ES)基础学习
### 1.采用倒排索引
+ 词条（term）: 按语义分成的词语
+ 文档（document）：每条数据就是一个文档

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436001042-2b74a8db-0bb1-45c2-8307-eb072b527e58.png)

### 2.es与mysql对比
> ES: 面向文档存储的，一条数据就是一个文档。
>
> 文档数据会被序列化json格式后存储在elasticsearch中
>

> 索引（index）：相同类型的文档的集合，如：订单信息（订单索引）、人员信息（人员索引）。
>
> 映射（mapping）: 索引中文档的约束，类似数据库中的表结构（Schema）.
>

**概念对比**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436001043-475bf8b2-120a-452a-90e1-d85f1af2ed6a.png)

**架构**

+ Mysql: 擅长事务类型操作；可以确保数据的安全和一致性。
+ Elasticsearch: 擅长海量数据的搜索、分析、计算。

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436000877-c6aeabc6-c318-493b-8a48-89f80a5aa277.png)

### 3.安装
#### 3.1.安装 es（部署单点es）
> 采用的elasticsearch7.12.1版本镜像
>

1. 创建网络

因为需要部署kibana容器，因此需要让es和kibana容器互联；这里先创建一个网络：

```plain
docker network create es-net
```

2. 加载镜像

```plain
docker load -i es.tar
```

3. 运行

运行docker命令，部署单点es：

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436001019-1cdc976d-14b5-4746-b643-9f36fc8e3f7a.png)

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436001072-aaae6ec6-6535-4b6a-b051-3e4826cab6f8.png)

#### 3.2.安装 kibana
> 采用的elasticsearch7.12.1版本镜像.
>
> 可以提供可视化界面，方便管理.
>

1. 加载镜像

```plain
docker load -i kibana.tar
```

2. 部署

运行docker命令，部署kibana:

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436001577-226329f3-1ef5-4ef6-9f66-279f9da519d9.png)

#### 3.3.安装IK分词器
> es在创建倒排索引时需要对文档进行分词；在搜索时需要对输入内容分词；
>
> 默认的分词规则对中文不太友好。**在处理中文分词，一般会使用IK分词器**
>
> <!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436001399-d456a337-8d03-40d7-94a0-5278e0a901bd.png)
>

> 测试：
>
> <!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436001493-1821f4f6-3a9c-4c7b-9b37-6abbbe4d8afb.png)
>

1. 安装ik插件（在线较慢，不推荐）

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436001464-3d556333-c46c-436c-9748-78bdc0e0a26d.png)

2. 离线安装ik插件（推荐）

1）查看数据卷目录

安装插件需要知道elasticsearch的plugins目录的位置，我们使用了数据卷的挂载，因此需要查看elasticsearch的数据卷目录，

使用命令查看：

```plain
docker volume inspect es-plugins
```

显示结果：

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436001994-487e0f39-4076-4705-8007-61d9f4e6c1a7.png)

2）解压缩、上传分词器安装包，重命名为ik

解压后，上传到数据卷的目录。

3）重启容器

```plain
docker restart es
```

```plain
docker logs -f es
```

    1. 测试

ik分词器2中模式：

    - **ik_smart : 智能切分、最少切分；粗粒度**
    - **ik_max_word : 最细切分；细粒度（耗内存）**

#### 3.4.IK分词器的拓展和停用词典
> 拓展新的词语
>
> 停用无意义的词
>

+ 修改ik分词器目录中的config目录中的ikAnalyzer.cfg.xml文件。
+ 拓展：在ext.dic的文件中添加想要拓展的词语
+ 停用：在stopword.dic的文件中添加想要停用的词语

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436001855-8fe82bb3-3a18-4512-a864-5662ae127aa8.png)

注意：需要重启es，才能起作用

### 4.操作索引库（类似数据库中的表）
+ mapping映射的属性
+ 索引库的CRUD

#### 4.1.mapping映射
mapping是对索引库中文档的约束，常见的mapping属性：

+ type： 字段数据类型，常见的类型： 
    - 字符串：text（可分词的文本）、keyword: （精确值，例如：品牌、国家、ip地址）
    - 数值：long、integer、sort、byte、double、float
    - 布尔：boolean
    - 日期：date
    - 对象：object
+ index：是否创建索引（倒排索引），默认为true。
+ analyzer：使用哪种分词器
+ properties：该字段的子字段

#### 4.2.索引库的CRUD
1. 创建索引库

ES中通过Restful请求操作索引库、文档。请求内容用哪个DSL语句来表示。

**创建索引库和mapping的DSL语法如下：  
**<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436001920-73f8042c-83fb-4352-a424-477bb13e2c79.png)

1. 查询、删除索引库

查看：`**GET /索引库名称**` 如：`**GET /heima**`

删除：`**DELETE /索引库名称**` 如：`**DELETE /heima**`

2. 修改索引库

==ES中一般是禁止修改索引库；==但是可以添加新的字段，语法如下：

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436001898-1dc1314e-c55a-4d0d-bb27-cb923b171c2b.png)

### 5.文档的操作（数据的操作）
1. 新增文档

DSL语法：<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436001949-78b14a42-73dd-41e0-8d68-2bc560930161.png)

2. 查询文档，DSL语法：  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436002202-91f6b9ee-bcee-4d63-98c6-1a5926aaceab.png)
3. 删除文档，DSL语法：   
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436002515-a3278255-c5dd-4d9a-9f65-8b8077d2fdf0.png)
4. 修改文档
    - 方式一：全量修改，会删除旧文档，添加新文档

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436002485-89ffa295-73c2-4b44-88b7-e2f0be6b58a3.png)

    - 方式二：增量修改，修改指定字段值

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436002454-4f84d244-780f-4f88-9223-2af49c4b494f.png)

### 6.RestClient操作索引库
> ES提供了各种不同语言的客户端。
>
> 这些客户端的本质就是组装DSL语句;
>
> 通过http请求发送给ES。
>
> **例如：JavaRestClient**
>

**通过案例学习**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436002696-8e2678d1-e975-4f20-8342-89d590d01423.png)

#### 6.1.分析数据结构，编写mapping
+ id：有点特殊，es中一般为`keyword`
+ 根据业务区分是否**分词和参与搜索**
+ 地理坐标：特殊字段，不是varchar，也不是daouble：

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436002480-07c2b8ad-3844-4773-98d2-b01c721cc6de.png)

+ （copy_to）解决多个字段搜索，效率又比较高：

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436002758-d90fcf77-8f04-4f8a-b969-b59477101ed5.png)

#### 6.2. 初始化JavaRestClient（选择RestHighLevelClient）
1. 引入es的RestHighLevelClient依赖：

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436002969-fa632b7f-ae61-4288-a6df-6a67f34991f1.png)

2. 覆盖默认的SpringBoot的ES版本：

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436002857-ed657b29-8c03-40ee-863e-e80c5b30fb53.png)

3. 初始化RestHighLevelClient：

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436002778-126a671a-e461-4732-98e3-e81c6fe194f5.png)

#### 6.3.创建索引库
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436003620-befcadbd-3214-4f9d-b14c-f279d9dceaf2.png)

#### 6.4.删除和判断索引库
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436003443-946346d5-464e-4282-8a45-a6f32003a4a8.png)

### 7.RestClient操作文档
+ 创建不同类型的请求对象
+ 准备参数（如果需要）
+ 发送请求

#### 7.1.新增文档
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436003269-4aa8239d-b0c0-4613-9bb6-3e6a52c76e5f.png)

#### 7.2.查询文档
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436003442-035edee2-6289-4ca0-9466-2361c12d7be8.png)

#### 7.3.更新文档
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436003700-2dba7d8e-806d-4939-a233-8e351417ca7a.png)

#### 7.4.删除文档
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436003980-0e92723b-c22d-4abc-8008-465601f6a70c.png)

#### 7.5.批量导入文档
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436004083-b2fe162d-0de8-48da-868a-61c33cc372e6.png)

---

---

## elasticsearch搜索功能
### 1.DSL查询分类和基本语法
#### 1.1.查询分类
> elasticsearch提供了基于JSON的DSL来定义查询。
>
> 常见的查询分类包括：
>
> + 查询所有：查询出所有数据，一般测试用。例如：match_all
> + 全文（full_text）检索查询：利用分词器对用户输入内容分词，然后如倒排索引库中匹配。如： 
>     - match_query
>     - multi_match_query
> + 精确查询：根据精确词条值查找数据，一般是查找keyword、数值、日期、boolean等类型字段。如： 
>     - ids
>     - range
>     - term（某个值==？）
> + 地理（geo）查询：根据经纬度查询。如： 
>     - geo_distance
>     - geo_bounding_box
> + 复合（compound）查询：可以将上述各种查询条件组合起来。合并查询条件。例如： 
>     - bool
>     - function_score
>

#### 1.2.查询的基本语法
**查询三要素**：查询类型、查询条件、条件值

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436003941-9a262e5c-d787-486a-be80-cb06c8ebf411.png)

#### 1.3.全文检索查询
**全文检索查询，会对用户输入的内容分词，常用于搜索框搜索。**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436004399-47cab2d7-4a43-4934-9003-894c9a872e4f.png)

#### 1.4.精确查询
**一般是查询keyword、数值、日期、boolean等类型字段。**

**所以一般不会被分词，常见的类型：**

+ term：根据词条值精确查找
+ range：根据值的范围查询

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436004160-759dcae9-603c-4de2-a3cd-e08fe1a995d4.png)

#### 1.5.地理查询
**根据经纬度查询，常见的使用场景：**

+ 携程：搜索我附近的酒店
+ 滴滴：搜索我附件的出租车
+ 微信：搜索我附近的人

常见的类型：

+ geo_bounding_box：查询geo_point值落在某个矩形范围的所有文档

**合适一定范围的所有信息，例如：地图找房**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436004799-0952d8fd-6dca-46df-9b00-0dd667cf0635.png)

+ geo_distance：查询到指定中心点小于某个距离值的所有文档

**附近的概念**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436004504-ae4b60dd-8e19-4eba-b7ef-4879363222df.png)

#### 1.6 复合（compound）查询
**复合查询可以将其它简单查询组合起来，实现更复杂的搜索逻辑，例如：**

> 1. function score
>
> 算分函数查询，可以控制文档相关性算分，控制文档排名。例如百度竞价
>
>     - **相关性算分**
>
> <!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436004805-6e6bc0ef-d2d4-4a64-9ebf-c2bc38bd86c8.png)  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436004564-335fd9c6-c769-438c-96b7-e6e5caead980.png)
>

> + **可以修改文档的相关性算分（query score）,根据新得到的算分排序。**
>
> <!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436005652-05d2e1e9-0009-4b4c-a5f8-7b4b04bff1ca.png)
>

> + **案例：**
>
> <!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436004960-777c8534-71d3-44b3-862f-eb538170e1f2.png)
>
> + **总结**
>
> <!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436005516-39b3bc78-6118-4b25-9544-8ca1799af771.png)
>

> 1. 复合查询 Boolean query
>
> 布尔查询是一个或多个查询子句的组合。组合方式有：
>
>     - must：必须匹配每个子查询，类似【与】【and】
>     - should：选择性匹配子查询，类似【或】【or】
>     - must_not：必须不匹配，不参与算分，类似【非】【!=】
>     - filter：必须匹配，但是不参与算分
>
> <!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436005411-0870fc8f-5603-4c34-94aa-4ed7562605c6.png)
>

> + **案例：**
>
> <!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436005311-639ff0fc-218d-455c-913f-d0213fc08faf.png)
>

### 2.搜索结果的处理
#### 2.1.排序
> elasticsearch 支持对搜索结果的排序，默认根据相关度算分（_score）来排序。
>
> 可以排序的字段类型有：
>
> + keyword类型
> + 数值类型
> + 地理坐标类型
> + 日期类型<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436005404-5438efeb-f23f-4166-afba-ea4c493e4e68.png)
>

#### 2.2.分页
> elasticsearch 默认情况下只返回top10的数据。
>
> 如果需要更多数据就需要修改分页参数：
>
> 通过修改from、size参数来控制返回的分页结果：
>
> <!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436005744-2213ca06-40ee-48bd-a4be-35062ab39c8b.png)
>

> 深度分页问题：
>
> ES是分布式的，所以会面临深度分页问题
>
> <!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436005979-f95d322e-3284-49c2-a196-18b67a8c086e.png)  
深度分页解决方案:
>
> + search after：分页时需要排序，原理是从上一次的排序值开始，查询下一页数据。推荐使用的方式。
> + scroll：原理将排序数据形成快照，保存在内存。官方已不推荐使用
>

> 总结：
>
> <!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436006041-33bfa6ee-367e-47c3-bb89-1b4a3ecfc6bb.png)
>

#### 2.3.高亮
> 高亮：就是在搜索结果中把关键字突出显示。
>
> 原理：
>
> + 将搜索结果中的关键字用标签标记出来
> + 在页面中给标签添加css样式
>
> 语法：<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436006165-50cc7cd0-a85a-474f-b271-7883121e9d6a.png)
>

> 注意：
>
> 高亮查询，默认情况下，ES搜索字段必须与高亮字段一致；
>
> 但是可通过require_field_match属性配置；
>
> <!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436006907-e339d808-40a8-443a-9110-0cb4d12c3789.png)
>

> 总结：
>
> <!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436006510-a4926210-544d-4e05-8645-5d5cad2200a9.png)
>

---

### 3.RestClient 查询文档
#### 3.1.快速入门
实现的3步法：

1. 准备request
2. 准备DSL
3. 发送请求

通过match_all来演示基本的API：

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436006473-1606cf6e-77cf-4055-8c58-bf66ed7ff809.png)

结果的解析：

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436007558-218a8adf-c426-4464-93a3-261fe79627ca.png)

总结：

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436006647-5ba36d52-c211-490a-b05e-b491a7e1e740.png)

#### 3.2.全文检索查询
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436007287-43b7492e-4c43-4c48-adfb-b48c07d80b5f.png)

#### 3.3.精确查询
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436006967-e0e7b44b-6617-4cd0-b2bd-c68920f72c2d.png)

#### 3.4.复合查询
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436007216-cb459c01-6069-479d-a19f-71e4b94c637e.png)

#### 3.5.排序、分页、高亮
1. 排序和分页

搜索结果的排序和分页是与query同级的参数：

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436007532-3dead41b-47c0-4609-80df-0b361c512783.png)

2. 高亮

> 高亮API包括请求DSL构建和结果解析2部分
>
> DSL构建：
>
> <!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436007456-2f0571f9-7e2b-4904-932b-b4421001ad82.png)
>

**实现：****但是结果并没有高亮**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436007765-ebc1decc-7ac9-47f9-aca6-29ca1bc69200.png)

**高亮结果解析：****高亮需要单独解析**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436008933-a3f9fe7f-cc89-40a7-a2db-e4f0c6835f9b.png)

---

---

## 黑马旅游案例
### 1.酒店的搜索和分页
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436008203-3f746b1b-7eaa-4f77-9e81-c724715c4342.png)

实现：

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436008478-9efbb5de-1850-4096-9cd9-5f70fca41bcc.png)

### 2.酒店结果的过滤
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436008251-9c8d1ba2-b93a-493d-bb95-0ec8b293daea.png)

分析设计：

+ 修改接收参数实体
+ 分析业务条件

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436008313-aaf9f480-37d0-4539-a75a-402eed2cf79b.png)

实现：

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436009285-eb0b4491-d731-4f37-8b15-232684f858c5.png)

### 3.我周边的酒店
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436009172-07382964-5d6b-4724-aeb2-71bf3200f06b.png)

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436009371-253d0fa4-9030-485f-9192-f6c79df5664e.png)

分析：

距离排序与普通字段排序的区别：

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436009435-29ccbb49-38b1-4944-82c5-594b91e1f9d3.png)

实现：

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436010046-a5e698c8-934e-4670-a581-a1023a412356.png)

### 4.酒店的竞价排名
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436010425-e45e3eba-3fcf-4634-b44a-cc605b1d2e07.png)

分析：

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436010225-a00bca26-c4bb-491e-bbd8-57f871031305.png)

实现：

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436010433-1de7831e-5d4a-418a-94b2-01759f1715af.png)

---

---

## 深入-数据聚合
### 1.聚合的种类
> 聚合（aggregations）可以实现对文档数据的统计、分析、运算。
>

**聚合常见的3个种类：**

+ 桶（bucket）聚合：用来对文档做分组；

不能是分词的字段；必须是keyword、数值、日期、boolean

    - termAggregation：按照文档字段值分组
    - date Histogram：按照日期阶梯分组，例如：一周为一组。
+ 度量（metric）聚合：用以计算一些值，比如：最大值、最小值、平均值。
    - avg：求平均值
    - max：求最大值
    - min：求最小值
    - stats：同时求max、min、avg、sum等
+ 管道（pipeline）聚合：其它聚合的结果为基础做聚合

### 2.DSL实现聚合
#### 2.1. 实现Bucket聚合
1. 聚合实现

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436010373-c211f6f1-ab49-40ea-abc2-785e7a04142e.png)

2. Bucket聚合-结果排序

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436010772-8b4a31b0-c80d-46fb-869e-6c67bade0a2a.png)

3. Bucket聚合-限定范围

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436010754-4869842a-0206-4431-96f3-bbfba5e77502.png)

4. 总结

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436010851-8b4f62ad-c501-455a-a368-727f099fe088.png)

#### 2.2.实现Metrics聚合
1. 聚合实现和排序

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436011080-975649c7-024f-49a4-a925-460832870694.png)

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436011163-6e5876e4-f574-4a29-87fe-75de3a45a0b8.png)

### 3.RestClient实现聚合
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436011461-415c9dbd-12d9-4978-8610-a34c19a6da72.png)

聚合结果的解析：

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436011953-9a38d76b-6055-4255-a6e9-d7aa6f5d98c9.png)

多条件聚合-案例：

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436012327-5e8a5759-469e-49dc-a68a-843cb53e989a.png)

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436012271-1b3e9700-1b51-4146-93ac-95023048f6fe.png)

---

---

## 深入-自动补全
### 1.安装拼音分词器
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436011693-0ea6690d-e4b8-4d44-94ed-99f0ae51eb38.png)

**安装拼音分词器：**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436012113-e112a7fd-82d7-49e1-965b-0c979709076b.png)

### 2.自定义分词器
> elasticsearch中分词器（analyzer）的组成包含三部分：
>
> + character filters：在tokenizer之前对文本进行处理。例如删除字符、替换字符。
> + tokenizer：将文本按照一定的规则切割成词条（term）。例如keyword，就是不分词；还有ik_smart。
> + tokenizer filter：将tokenizer输出的词条做进一步处理；例如大小写转换、同义词处理、拼音处理等。
>
> <!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436012177-caf8c5c7-541b-4a3e-9029-461c0a8ea2ef.png)
>

**语法：**

必须在创建索引库时，进行自定义分词器的设置：

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436012782-110ed85b-9559-4909-a25a-9f93ae410d22.png)

注意事项-同义词：

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436012765-47dd5e1b-57e7-4abf-901d-8bdeeef22d06.png)

注意事项-同义词-解决方案：

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436012797-d53c0e9b-2bf5-4e1c-94fb-82bf13b4619e.png)

**总结：**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436012840-900bc715-498c-4f88-a3a5-07bc288d1eea.png)

### 3.自动补全查询
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436013174-070ebf60-9dee-4834-9ec7-ac9f0a964aee.png)  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436013152-8f07b8ac-9c12-4a5a-a10a-f1d85f915e76.png)

**语法：**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436013277-4b678ddc-f3f0-4edc-9682-54a8d655bb58.png)

### 4.实现酒店搜索框自动补全
案例分析：

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436013394-e248f80c-8cac-497e-a128-9c4aded0a4d0.png)

验证：

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436013404-f06a361e-895f-499a-bd22-ae8010b2aaea.png)

### 5.RestClient实现自动补全查询
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436014068-187f01db-f029-4745-a58e-48750910475f.png)

**结果解析：**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436014075-e329f09c-14b6-4ccd-8fde-3dde56ce97ca.png)

### 6. 案例实现搜索框自动补全查询
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436014097-32b962d2-cb40-498b-817b-3b27a9f39c82.png)

---

---

## 深入-数据同步
### 1.数据同步思路分析
分析：

> elasticsearch中的数据来自于数据库，因此数据发生改变时，ES也必须跟着改变，
>
> 这个就是es与数据库之间的**数据同步**。
>
> <!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436013962-5363bea8-2c4a-4575-9ac0-25074d458494.png)
>

**方案一：**同步调用

依次操作、数据耦合

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436014297-443dba33-9703-49df-a8b8-15f6aee37845.png)

**方案二：**异步通知

比较推荐、依赖MQ、实现复杂度提升

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436014566-b6dd7cac-7c40-4c54-a8c4-c6e75f052cdc.png)

**方案三：**监听binlog

需开启数据库的binlog，数据库压力增加，增加了中间件，复杂度高

binlog：mysql可开启binlog,binlog可记录数据的变动

canal：可监听binlog

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436014713-235aa3e9-4fa7-48cc-85e1-dd6c7b6f8fe5.png)

总结：

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436014675-b5151c14-6336-43a8-a8ef-4702c255cd98.png)

### 2.MQ实现数据同步（rabbitmq）
案例：

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436014874-5ba89fc6-42f0-4722-969f-f8241ac4fe16.png)

实现和分析：

消息模型：

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436014794-a32fe58d-a70c-4757-814f-0e8ee305538f.png)

实现步骤：

+ 声明交换机和队列以及key的绑定(一般在消费者端)
+ 发送MQ消息
+ 监听MQ消息
+ 测试

---

---

## 深入-集群
### 1.搭建ES集群
> 问题：
>
> 单机的ES做数据存储，必然2个问题：
>
> + 海量数据存储问题：将索引库从逻辑上拆分为N个分片（shard）,存储到多个节点
> + 单点故障问题：将分片数据在不同节点备份（replica）
>
> <!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436015040-47d5ede3-be0e-4afe-bd23-bfd168d28f5d.png)
>

**搭建集群：**

> 计划利用docker模拟3个es的节点。
>
> 部署es集群可以直接使用docker-compose来完成。
>
> <!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436015328-fc7ffcf7-c459-41d6-80c5-8f2d0df3a178.png)
>

> <!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436015433-5ce0494c-3537-4ea7-95c8-606e269b45c6.png)
>

> <!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436015080-3c7c9bd5-fb9f-411b-9644-34a3ead3f818.png)
>

> <!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436015282-e28db2d5-1a2a-4db2-904b-c5091031be47.png)
>

**集群状态的监控：cerebro**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436015465-bb686407-6964-48d9-ad46-753a246a28d0.png)

**分片和备份：**

+ kibana：

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436015469-4cb01aac-08e9-4807-8465-9abbb1710e5c.png)

+ cerebro：

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436015786-a24f777b-0d8b-48fb-8fa4-f7948304daac.png)

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436015672-3ac489e1-2124-4d98-99da-5b4f9f83ea43.png)

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436016073-e6b5a5f7-bb1c-4db2-a2f2-c520bf347b7b.png)

### 2.集群的职责和脑裂
#### 2.1.ES集群的节点角色
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436016046-ab83e0d5-3bfc-458d-b76c-415fe13d09e3.png)

**一个典型的ES集群一定是把每个节点的职责拆分成不同的节点，干不同的事！！**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436016065-1c8b29a1-8694-4ae2-b58a-d27cf869d149.png)

#### 2.2.ES集群的脑裂
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436016318-e733eb98-48d7-494a-8868-516b92891508.png)

#### 2.3.总结
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436016311-6421fab5-d039-4137-9914-76efc84dc2ad.png)

### 3.分布式新增和查询流程
#### 3.1.分布式存储
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436016575-c944d2c6-542c-4210-9627-9819036f5fdf.png)

#### 3.2.分布式新增
**新增文档流程：**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436016517-dd54014a-cd5e-4474-83a0-136cf1f26f83.png)

#### 3.3.分布式查询
**查询文档流程：**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436016643-d79295a2-a26d-4ac3-8887-bca4108b3e90.png)

### 4.ES集群的故障转移
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436016935-1f587c45-9a2e-42e1-8f26-5eab7bc97eaa.png)

**当节点恢复后，数据也会重新迁移回来！！**

总结：

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762436016758-969a4ba5-c3bb-41e4-8a10-072ff77102d5.png)







