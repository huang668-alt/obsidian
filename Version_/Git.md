## 1、Git概述
Git是一个免费的、开源的分布式版本控制系统，可以快速高效地处理从小型到大型的各种项目。Git 易于学习，占地面积小，性能极快。 它具有廉价的本地库，方便的暂存区域和多个工作流分支等特性。其性能优于 Subversion、CVS、Perforce 和 ClearCase 等版本控制工具。

### 1.1 何为版本控制
版本控制是一种记录文件内容变化，以便将来查阅特定版本修订情况的系统。  
版本控制其实最重要的是可以记录文件修改历史记录，从而让用户能够查看历史版本，方便版本切换。  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078732695-6dc16e19-5014-49dc-a30d-0e0788164372.png)

### 1.2 为什么需要版本控制
个人开发过渡到团队协作  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078732686-c4d3a458-0808-4141-943d-2ebc6d15e0e5.png)

### 1.3 版本控制工具
+ 集中式版本控制工具  
CVS、SVN(Subversion)、VSS……  
集中化的版本控制系统诸如 CVS、SVN 等，都有一个单一的集中管理的服务器，保存所有文件的修订版本，而协同工作的人们都通过客户端连到这台服务器，取出最新的文件或者提交更新。多年以来，这已成为版本控制系统的标准做法。  
这种做法带来了许多好处，每个人都可以在一定程度上看到项目中的其他人正在做些什么。而管理员也可以轻松掌控每个开发者的权限，并且管理一个集中化的版本控制系统，要远比在各个客户端上维护本地数据库来得轻松容易。  
事分两面，有好有坏。这么做显而易见的缺点是中央服务器的单点故障。如果服务器宕机一小时，那么在这一小时内，谁都无法提交更新，也就无法协同工作。  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078732702-c49cda11-ffdd-4cca-8ab2-aa9dbe2ad47a.png)
+ 分布式版本控制工具  
Git、Mercurial、Bazaar、Darcs……  
像 Git 这种分布式版本控制工具，客户端提取的不是最新版本的文件快照，而是把代码仓库完整地镜像下来（本地库）。这样任何一处协同工作用的文件发生故障，事后都可以用其他客户端的本地仓库进行恢复。因为每个客户端的每一次文件提取操作，实际上都是一次对整个文件仓库的完整备份。分布式的版本控制系统出现之后,解决了集中式版本控制系统的缺陷:
    1. 服务器断网的情况下也可以进行开发（因为版本控制是在本地进行的）
    2. 每个客户端保存的也都是整个完整的项目（包含历史记录，更加安全）  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078732697-bfdd5390-34d5-4fd4-82a9-da9b9d571655.png)

### 1.4 Git简史
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078732728-a4d6264b-3f69-4bd5-95a6-1bc020b9baf3.png)

### 1.5 Git工作机制
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078735212-7c2b44fa-4cd8-49b9-9c35-da8e4e63b3a9.png)

### 1.6 Git和代码托管中心
代码托管中心是基于网络服务器的远程代码仓库，一般我们简单称为远程库。  
➢ 局域网  
✓ GitLab  
➢ 互联网  
✓ GitHub（外网）  
✓ Gitee 码云（国内网站）

## 2、Git安装
官网地址： [https://git-scm.com/](https://git-scm.com/)  
1.查看 GNU 协议，可以直接点击下一步。  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078733299-63acb7dd-19ae-45b9-9f5f-b322a3b4fdf5.png)  
2.选择 Git 安装位置，要求是非中文并且没有空格的目录，然后下一步。  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078733318-9ed694f9-e99b-4adb-966e-4e4379b2fcb1.png)  
3.Git 选项配置，推荐默认设置，然后下一步。  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078735333-cfacb151-88b0-4bd4-a0c4-684e52ded07c.png)  
4.Git 安装目录名，不用修改，直接点击下一步。  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078735385-99d012d3-bded-4923-b8d3-fe0d604eb126.png)  
5.Git 的默认编辑器，建议使用默认的 Vim 编辑器，然后点击下一步。

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078733720-5599f70f-333c-43e1-8490-0547f771f145.png)  
6.默认分支名设置，选择让 Git 决定，分支名默认为 master，下一步。  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078733764-450ecd53-cb49-4d7f-9dea-179278fef2e2.png)  
7.修改 Git 的环境变量，选第一个，不修改环境变量，只在 Git Bash 里使用 Git。  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078734091-224391f3-94c0-425d-a49e-9e23c8325f04.png)  
8.选择后台客户端连接协议，选默认值 OpenSSL，然后下一步。  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078736172-5321199b-e2f4-4ad0-9152-f45574a517ea.png)  
9.配置 Git 文件的行末换行符，Windows 使用 CRLF，Linux 使用 LF，选择第一个自动转换，然后继续下一步。  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078734500-e6f9845f-d831-4f6f-bbc9-b0f323360d33.png)  
10.选择 Git 终端类型，选择默认的 Git Bash 终端，然后继续下一步。  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078734864-2298a051-1923-42ef-b9b6-bd7e1b854c5f.png)  
10.选择 Git pull 合并的模式，选择默认，然后下一步。  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078735211-f82e4232-8bc4-40cc-ab38-20ade78452b6.png)  
11.选择 Git 的凭据管理器，选择默认的跨平台的凭据管理器，然后下一步。  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078735535-03fa616f-d08d-449d-8e66-76e01b71c263.png)  
12.其他配置，选择默认设置，然后下一步。  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078735611-a50dccf7-913e-417c-b82c-33170d483d04.png)  
13.实验室功能，技术还不成熟，有已知的 bug，不要勾选，然后点击右下角的 Install按钮，开始安装 Git。  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078735761-1ed29f17-e645-48f7-9a24-24a5f5f8334d.png)  
14.点击 Finsh 按钮，Git 安装成功！  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078735814-7863c620-bea6-4b23-9f1b-5975a30cddfd.png)  
15.右键任意位置，在右键菜单里选择 Git Bash Here 即可打开 Git Bash 命令行终端。  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078735980-a6b453a8-591a-4c04-a533-0dfc40beda39.png)  
16.在 Git Bash 终端里输入 git --version 查看 git 版本，如图所示，说明 Git 安装成功。  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078736100-c40e4a06-7a5c-4ab7-a875-aab74039bfdc.png)

## 3、Git常用命令
| 命令名称 | 作用 |
| --- | --- |
| git config --global user.name | 用户名 设置用户签名 |
| git config --global user.email | 邮箱 设置用户签名 |
| **git init** | 初始化本地库 |
| **git status** | 查看本地库状态 |
| **git add** | 文件名 添加到暂存区 |
| **git commit -m** | “日志信息” 文件名 提交到本地库 |
| **git reflog** | 查看历史记录 |
| **git reset --hard** | 版本号 版本穿梭 |


### 3.1 设置用户签名
1 ）基本语法

```plain
git config --global user.name 用户名
git config --global user.email 邮箱
```

2 ）案例实操  
全局范围的签名设置：  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078736341-04fd18d0-40ed-4fec-abce-ef337b0fcb04.png)  
说明：  
签名的作用是区分不同操作者身份。用户的签名信息在每一个版本的提交信息中能够看到，以此确认本次提交是谁做的。Git 首次安装必须设置一下用户签名，否则无法提交代码。  
`※注意`：这里设置用户签名和将来登录 GitHub（或其他代码托管中心）的账号没有任  
何关系。

### 3.2 初始化本地库
1）基本语法

```plain
git init
```

2 ）案例实操  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078736368-446a34fe-9cc7-45d5-bcb6-a51ce356d158.png)  
3 ）结果查看  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078736387-fc52057b-b4fe-4688-a8e7-c17438e58edb.png)

### 3.3 查看本地库状态
1） 基本语法

```plain
git status
```

2）案例实操  
1、首次查看（工作区无任何文件）  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078736879-b054015c-4a77-4d2c-86d0-361562dc90ac.png)  
2、 新增文件（ hello.txt ）  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078736601-d6a78b5d-df68-457b-9c77-54c553e017f6.png)  
3、再次查看（检测未被追踪到的文件）  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078736825-9d350682-2e57-472e-bd5a-87533a6e2b10.png)

### 3.4 添加暂存区
1）基本语法

```plain
git add 文件名
```

2）案例实操  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078736847-13091e74-1c0f-40de-a434-465475bb59b1.png)  
查看状态，检测到暂存区有新文件

### 3.5 提交到本地库
1）基本语法

```plain
git commit -m "日志信息" 文件名
```

2）案例实操  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078736861-73d6d469-16ed-415d-ae16-d259093ebcdf.png)  
查看状态，没有文件需要提交  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078737046-88e92102-a646-4953-b6db-898081dc9785.png)

### 3.6 修改文件
1、修改hello.txt  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078737287-b375f306-1bca-47d9-8ee8-aa122f28ae03.png)  
2、查看状态，检测到工作区有文件被修改  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078737274-e30048b0-2059-4274-968a-718c944d0cfc.png)  
3、将修改的文件再次添加到暂存区  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078737237-5b1214ff-e6ce-44bb-abb8-474cf6a83a2d.png)

4、将修改的文件再次提交  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078737340-4dccf698-4787-49d7-a3b6-2c2814f39fba.png)

### 3.7 历史版本
**3.7.1 查看历史版本**  
1）基本语法

```plain
git reflog   查看版本信息
git log      查看详细版本信息
```

2）案例实操  
git reflog  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078737479-7c6d59c3-4566-4c4c-ab64-d2f68af10ab1.png)  
git log  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078737695-0f9c60b5-9571-4814-9fd5-51aa7152f5fb.png)  
**3.7.2 版本穿梭**  
1）基本语法

```plain
git reset --hard 版本号
```

2）案例实操  
1、可以看出现在指向第二次提交的`6b1dc53`版本  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078737704-b3d4ea99-190c-4644-97af-5f830ce3de43.png)  
2、切换到第一次提交的 `5af531f`版本  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078737700-01feac86-da61-4382-8511-975b8e3e4bff.png)  
3、查看hello.txt，发现内容已经更改  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078737705-d2261f4a-f23e-455f-8bb2-a0251b3287af.png)  
4、查看日志，发现指向第一次提交的 `5af531f` 版本  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078737919-2838e301-0cd6-4079-924f-fff03c570448.png)

## 4、Git的分支操作
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078738133-72adb239-5b0d-40c1-94aa-5e1ffed1b0d5.png)

### 4.1 什么是分支
在版本控制过程中，同时推进多个任务，为每个任务，我们就可以创建每个任务的单独分支。使用分支意味着程序员可以把自己的工作从开发主线上分离开来，开发自己分支的时候，不会影响主线分支的运行。对于初学者而言，分支可以简单理解为副本，一个分支就是一个单独的副本。（分支底层其实也是指针的引用）  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078738128-82e8a089-7843-4ffb-86cd-481cbb56f018.png)

### 4.2 分支的好处
同时并行推进多个功能开发，提高开发效率。  
各个分支在开发过程中，如果某一个分支开发失败，不会对其他分支有任何影响。失败  
的分支删除重新开始即可。

### 4.3 分支的操作
| 命令名称 | 作用 |
| --- | --- |
| git branch 分支名 | 创建分支 |
| git branch -v | 查看分支 |
| git checkout 分支名 | 切换分支 |
| git merge 分支名 | 把指定的分支合并到当前分支上 |


**4.3.1 查看分支**  
1）基本语法

```plain
git branch -v
```

2 ）案例实操  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078738144-83a2d567-f15c-47ca-bcc3-3f6c2d0ebfec.png)  
*代表当前所在分区

**4.3.2 创建分支**  
1）基本语法

```plain
git branch 分支名
```

2 ）案例实操  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078738121-3d8da442-7ccb-4b40-a242-8603dc80107e.png)  
**4.3.3 修改分支**  
1、在`master`分支上做修改  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078738322-205faaba-f7a2-48b0-a4c5-251a50760466.png)  
2、添加至暂存区、提交到本地库  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078738476-e5227250-0ad1-4e21-a385-f85d100c2087.png)  
3、hot-fix 分支并未做任何改变、master 分支已更新为最新一次提交的版本  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078738535-6fac3761-056b-403c-9b1b-9564329daa54.png)  
**4.3.4 切换分支**

1）基本语法

```plain
git checkout 分支名
```

2 ）案例实操  
1、切换到hot-fix分支  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078738571-b688ecf6-f9ee-4a69-9512-a101be3f32dc.png)  
2、查看 hot-fix 分支上的文件内容发现与 master 分支上的内容不同  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078738565-2618fa95-b2c8-4fb3-b35c-8dfe9ceae960.png)  
3、在 hot-fix 分支上做修改  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078738731-1711f374-1133-45a2-adb3-3578189f9523.png)  
4、添加到暂存区、提交到工作区  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078738861-8fd104b1-12f9-4f43-8263-20a888e32b48.png)  
**4.3.5 合并分支**

1）基本语法

```plain
git merge 分支名
```

2 ）案例实操 在 master 分支上合并 hot-fix 分支  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078738925-026249b7-1042-4c69-8ca9-79780833dda6.png)  
**4.3.6 产生冲突**

冲突产生的表现：后面状态为 `MERGING`  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078738935-e09b4259-7ffd-462c-8d8e-ac2cffd52c4f.png)  
查看冲突文件hello.txt  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078739021-8eb846d8-ddb5-4222-9d1c-5f0b196e8fbb.png)  
冲突产生的原因：  
合并分支时，两个分支在`同一个文件的同一个位置`有两套完全不同的修改。Git 无法替  
我们决定使用哪一个。必须`人为决定`新代码内容。  
查看状态（检测到有文件有两处修改）  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078739191-e1dfb661-7148-4fee-b8e7-0d83f53709c7.png)  
**4.3.7 解决冲突**  
1）编辑有冲突的文件，删除特殊符号，决定要使用的内容  
特殊符号：`<<<<<<< HEAD` 当前分支的代码 `=======` 合并过来的代码 `>>>>>>> hot-fix`  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078739399-6b6b82ee-6120-4c8e-82d4-f646936987f8.png)  
2）添加到暂存区

3）执行提交（注意：此时使用 git commit 命令时不能带文件名）  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078739377-202ebc7b-bc12-4de5-80c3-326682fd8fb6.png)  
发现后面 MERGING 消失，变为正常

## 5、Git团队协作机制
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078739394-cfdf846f-ec8b-42c4-9864-1d84309511ca.png)  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078739520-e5066135-b83c-4fb2-a5e9-38b021c82cdf.png)

## 6、GitHub操作
GitHub 网址：[https://github.com/](https://github.com/)

### 6.1 创建远程仓库
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078739634-7a1c1e45-2c17-4f68-9688-f16d546c6c31.png)  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078739822-ff473028-ab65-40d8-b255-a1d3f59811cd.png)

### 6.2 远程仓库操作
| 命令名称 | 作用 |
| --- | --- |
| git remote -v | 查看当前所有远程地址别名 |
| git remote add 别名 远程地址 | 起别名 |
| git push 别名 分支 | 推送本地分支上的内容到远程仓库 |
| git clone 远程地址 | 将远程仓库的内容克隆到本地 |
| git pull 远程库地址别名 远程分支名 | 将远程仓库对于分支最新内容拉下来后与当前本地分支直接合并 |


**6.2.1 创建远程仓库别名**  
1） 基本语法

```plain
git remote -v 查看当前所有远程地址别名
git remote add 别名 远程地址
```

2）实操案例  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078739870-80593d1b-f6a9-4d61-8896-67f07a5d21c2.png)  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078739911-02438988-4f3b-4d57-b367-9db15e99dbe5.png)  
**6.2.2 推送本地分支到远程仓库**  
1） 基本语法

```plain
git push 别名 分支
```

2）实操案例  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078740036-4a6d9f6d-761a-420d-9f11-3b0ef966c4b4.png)  
**6.2.3 克隆远程仓库到本地**  
1） 基本语法

```plain
git clone 远程地址
```

2）实操案例  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078740043-0644c2bd-423e-4fb0-8197-533dbe4186f5.png)  
小结：clone 会做如下操作。1、拉取代码。2、初始化本地仓库。3、创建别名

**6.2.4 邀请加入团队**  
1） 选择邀请合作者  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078740272-3279d3b8-084a-4f53-85aa-d66b63fdb27f.png)  
2）填入想要合作的人  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078740487-079a63db-ca9b-4515-aa0e-8009f8f94b13.png)  
3） 复 制 地 址 并 通 过 微 信 钉 钉 等 方 式 发 送 给 该 用 户 ， 复 制 内 容 如 下 ：  
[https://github.com/atguiguyueyue/git-shTest/invitations](https://github.com/atguiguyueyue/git-shTest/invitations)  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078740455-f0b2fa24-6feb-480d-ba78-87f8c3b9e8a6.png)  
4）在 atguigulinghuchong 这个账号中的地址栏复制收到邀请的链接 ，点击接受邀请  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078740563-a8ee86e2-d984-40ab-a550-b9dc2a3293cc.png)  
5）在成功之后可以在 atguigulinghuchong 这个账号上看到 git-Test 的远程仓库。  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078740615-548780ec-48c0-4da1-a314-2d90adb86d25.png)  
6 ）令狐冲可以修改内容并 push 到远程仓库。  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078740693-0f90f1a5-7378-49d9-86b6-06c1cbd2856b.png)  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078740868-3898a9ad-e515-4509-af18-c97ef6896447.png)  
7）回到 atguiguyueyue的GitHub 远程仓库中可以看到，最后一次是 lhc 提交的  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078740926-efd2daea-72f4-4c22-be51-204a663152dc.png)  
**6.2.5 拉取远程库内容**  
1） 基本语法

```plain
git pull 远程库地址别名 远程分支名
```

2）实操案例  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078740954-04c83e39-38a2-4609-8917-bc2828fdb106.png)

### 6.3 跨团队协作
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078740992-7a59423c-6b17-45e6-b63c-d3239b850408.png)  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078741122-a939bdc0-02fa-4222-be7c-482e846d5a9b.png)  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078741553-751c978b-c8c4-42b8-9905-f3904698c4a9.png)  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078741546-02c43810-fd77-4834-b444-39ecfebe8941.png)  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078741563-3552ba02-05d9-46b2-9f46-cbcd256ad5e5.png)  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078741559-b0aa0f34-906c-4a18-b902-1098c6d9cc8e.png)  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078741812-34aba394-95f5-4fa5-b6cc-731b9aa58677.png)  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078742054-ccf0647f-865c-434e-9451-949aa9e92314.png)  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078742026-8af9fc53-26d2-46c6-8eab-68f98b7354b4.png)  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078742060-507d447e-579a-4b08-b5c6-36e71071918e.png)

### 6.4 SSH免密登录
我们可以看到远程仓库中还有一个 SSH 的地址，因此我们也可以使用 SSH 进行访问。  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078742125-d171812c-38c5-4ba5-8e5b-bd8a92554e4c.png)  
具体操作如下：  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078742371-3cd16d33-d75e-4393-bda8-8d32430ae665.png)<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078742375-cb2f1c58-0137-448d-b2de-cb9ec24a8cf6.png)  
复制 id_rsa.pub 文件内容，登录 GitHub，点击用户头像→Settings→SSH and GPG keys  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078742504-88500963-1bc0-4772-8989-c3293ad575f3.png)  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078742486-80417f4f-b8f7-4cb7-8622-9a0ca0b22bc3.png)  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078742562-a653a229-1f32-409e-98b1-d02c7b8b3d3e.png)  
接下来再往远程仓库 push 东西的时候使用 SSH 连接就不需要登录了。

## 7、IDEA集成Git
### 7.1 配置Git忽略文件
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078742819-125c5bfa-46d9-4ce2-9ad2-f3fb98e1be6f.png)  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078742868-696cf01e-f40e-483a-a3fd-bfe886b44e5a.png)  
问题 1: 为什么要忽略他们？  
答：与项目的实际功能无关，不参与服务器上部署运行。把它们忽略掉能够屏蔽 IDE工具之间的差异。

问题 2 ：怎么忽略？  
1）创建忽略规则文件 xxxx.ignore（前缀名随便起，建议是 `git.ignore`）  
这个文件的存放位置原则上在哪里都可以，为了便于让~/.gitconfig 文件引用，建议也放在用户家目录下  
git.ignore 文件模版内容如下：

```plain
# Compiled class file
*.class

# Log file
*.log

# BlueJ files
*.ctxt

# Mobile Tools for Java (J2ME)
.mtj.tmp/

# Package Files #
*.jar
*.war
*.nar
*.ear
*.zip
*.tar.gz
*.rar

# virtual machine crash logs, seehttp://www.java.com/en/download/help/error_hotspot.xml
hs_err_pid*

.classpath
.project
.settings
target
.idea
*.iml
```

2）在.gitconfig 文件中引用忽略配置文件（此文件在 Windows 的家目录中）

```plain
[user]
    name = Layne
    email = Layne@atguigu.com
[core]
    excludesfile = C:/Users/asus/git.ignore
注意：这里要使用“正斜线（/）”，不要使用“反斜线（\）”
```

### 7.2 定位Git程序
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078742917-7749327d-181e-4f18-8efa-64df0aa1b950.png)

### 7.3 初始化本地库
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078742957-aa03ce8b-fb4d-40b6-a4ce-c5efa4b4a1fb.png)  
选择要创建 Git 本地仓库的工程。  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078743038-81ac22b2-7049-44bd-8b21-41bb3d24e85b.png)

### 7.4 添加到暂存区
右键点击项目选择 Git -> Add 将项目添加到暂存区。  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078743254-0b7650d2-f2a2-4f11-8f4a-581c3be071c3.png)

### 7.5 提交到本地库
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078743288-c33b683c-a139-4c63-9ab3-4e0a13b7337f.png)

### 7.6 切换版本
在 IDEA 的左下角，点击 Version Control，然后点击 Log 查看版本  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078743338-ec0652a4-cd96-4564-ad45-631a486dbe91.png)  
右键选择要切换的版本，然后在菜单里点击 Checkout Revision。  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078743396-d3ea1a00-e770-42b2-a7a1-362294311e55.png)

### 7.7 创建分支
选择 Git，在 Repository 里面，点击 Branches 按钮。  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078743514-73ca66c0-129c-4e69-bb26-e6c34e4bd908.png)  
在弹出的 Git Branches 框里，点击 New Branch 按钮。  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078743817-be01c6f3-1fb6-4ef5-8636-b7765edc9dfa.png)填写分支名称，创建 hot-fix 分支。  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078743847-8591ccb3-b66e-4640-841d-a07ecdb26151.png)  
然后再 IDEA 的右下角看到 hot-fix，说明分支创建成功，并且当前已经切换成 hot-fix 分支  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078743853-e60c792e-276f-48d4-bf6a-f6c7b679ceaa.png)

### 7.8 切换分支
在 IDEA 窗口的右下角，切换到 master 分支。  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078743884-93b392b5-5f80-4609-a736-b0f3d890fbf7.png)  
然后在 IDEA 窗口的右下角看到了 master，说明 master 分支切换成功。  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078743983-44b2a320-6654-4c53-b52d-348fa298a839.png)

### 7.9 合并分支
在 IDEA 窗口的右下角，将 hot-fix 分支合并到当前 master 分支。  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078744255-6170221d-6645-4ba9-b9d5-011e3bfef5d5.png)  
如果代码没有冲突，分支直接合并成功，分支合并成功以后，代码自动提交，无需手动提交本地库。  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078744252-5140c688-16cb-4cc2-9716-c611b79a9d42.png)  
如果代码没有冲突，分支直接合并成功，分支合并成功以后，代码自动提交，无需手动提交本地库。  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078744283-4c4cd6ab-cac0-4c95-92c3-0eb45e268751.png)

### 7.10 解决冲突
如图所示，如果 master 分支和 hot-fix 分支都修改了代码，在合并分支的时候就会发生冲突。  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078744315-5dc97ead-ed8f-4fe3-9bf2-fb7ed2a1c5d7.png)  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078744552-9b05c644-bc3e-4f79-b69b-377893405fab.png)  
我们现在站在 master 分支上合并 hot-fix 分支，就会发生代码冲突。  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078744715-b13da241-9e65-468e-a200-3e9da109ab35.png)  
点击 Conflicts 框里的 Merge 按钮，进行手动合并代码。  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078744802-2bcd8a92-9f40-4b1e-960e-3986dc53327e.png)  
手动合并完代码以后，点击右下角的 Apply 按钮。  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078744786-03f8cfb7-c056-4cb1-b49f-27718f6bfa78.png)  
代码冲突解决，自动提交本地库。  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078744782-410505b2-03c5-4fbb-845d-5e175d93ae7a.png)

## 8、IDEA集成GitHub
### 8.1 设置 GitHub 账号
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078745023-af54ab35-f53b-40ef-87c2-55e874ebaf14.png)  
如果出现 401 等情况连接不上的，是因为网络原因，可以使用以下方式连接：

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078745191-b0a4759c-3b2e-43b2-86da-4a9ff1a56d45.png)  
然后去 GitHub 账户上设置 token。  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078745247-72f83618-3159-4f9d-a673-2e11a1099fd1.png)  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078745309-aba54079-478d-487f-aef7-ec39bf210af6.png)  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078745406-5d2148b8-b242-43ce-bdcc-f1fa9f1abfc7.png)  
点击生成 token。  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078745454-43e71e49-a664-4eac-95a2-aa05e3ca60cc.png)  
复制红框中的字符串到 idea 中。  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078745532-63fbef3d-7029-4c2d-895c-c3b4c2c4f20b.png)  
点击登录。  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078745711-0fcb5b18-c40e-463a-8f5e-ea8ec2ea5994.png)

### 8.2 分享工程到 GitHub
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078745751-64eedecf-0470-41d7-bf4f-ab21d0deb684.png)  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078745794-3ccc09f7-3e52-46cf-9fd2-c018b31acc5a.png)  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078745833-f119189c-6a4d-49b4-b1df-666ded516c72.png)  
来到 GitHub 中发现已经帮我们创建好了 gitTest 的远程仓库。  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078746074-aab90e6f-9d36-4df1-8278-2931c2f7fc04.png)

### 8.3 push 推送本地库到远程库
右键点击项目，可以将当前分支的内容 push 到 GitHub 的远程仓库中。  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078746090-030a479e-d20d-491d-a795-01a70c15110f.png)  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078746166-3b1ceb37-224c-4811-9a5d-e6dd5ecbd320.png)  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078746166-b2dec62d-e9e5-4e7c-82ac-b6e582145c86.png)  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078746216-6ee2e939-0fca-42a9-8dee-998b1ff80664.png)  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078746450-966bb998-7b82-4086-9abb-2603ab5ef735.png)  
注意：push 是将本地库代码推送到远程库，如果本地库代码跟远程库代码版本不一致，push 的操作是会被拒绝的。也就是说，要想 push 成功，一定要保证本地库的版本要比远程库的版本高！因此一个成熟的程序员在动手改本地代码之前，一定会先检查下远程库跟本地代码的区别！如果本地的代码版本已经落后，切记要先 pull 拉取一下远程库的代码，将本地代码更新到最新以后，然后再修改，提交，推送！

### 8.4 pull 拉取远程库到本地库
右键点击项目，可以将远程仓库的内容 pull 到本地仓库。  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078746498-dd0a2bc2-6397-4215-bebe-122101a2218e.png)  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078746506-5a96c91b-74d5-4e00-9bd4-22ecdef622b2.png)  
注意：pull 是拉取远端仓库代码到本地，如果远程库代码和本地库代码不一致，会自动合并，如果自动合并失败，还会涉及到手动解决冲突的问题。

### 8.5 clone 克隆远程库到本地
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078746585-2196fadf-13bc-42a2-8ec5-756a96394d85.png)  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078746794-3fa2e35d-1106-408f-885f-31e072db0d19.png)  
为 clone 下来的项目创建一个工程，然后点击 Next。  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078746840-940502ec-5102-4a5f-bc6f-7464b64a1ce5.png)  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078746838-f48fc10b-7f40-4df6-a6b5-895f9162e858.png)  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078746894-b57673b5-4321-4769-b806-d2d2ffb06ab4.png)  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078747062-ac13c32d-e54c-4e8b-8cf6-6f20d106c5a4.png)

## 9、国内代码托管中心-码云
### 9.1 简介
众所周知，GitHub 服务器在国外，使用 GitHub 作为项目托管网站，如果网速不好的话，严重影响使用体验，甚至会出现登录不上的情况。针对这个情况，大家也可以使用国内的项目托管网站-码云。  
码云是开源中国推出的基于 Git 的代码托管服务中心，网址是 [https://gitee.com/](https://gitee.com/) ，使用方式跟 GitHub 一样，而且它还是一个中文网站，如果你英文不是很好它是最好的选择。

### 9.2 码云帐号注册和登录
进入码云官网地址：[https://gitee.com/，点击注册](https://gitee.com/%EF%BC%8C%E7%82%B9%E5%87%BB%E6%B3%A8%E5%86%8C) Gitee，输入个人信息，进行注册即可。

### 9.3 码云创建远程库
点击首页右上角的加号，选择下面的新建仓库  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078747160-25f8c51f-e4b6-49cf-bc8d-e6b3a4d8dce0.png)  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078747188-5d9a0db0-ba7f-4ca0-8882-e60d88064741.png)  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078747208-0a025775-eda1-411e-bed6-9f1aa6699b1b.png)  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078747268-a92e9918-7bf0-435a-82b5-841e940e92ba.png)  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078747654-37d51890-f13e-46f5-8dc8-5bf31b24e48a.png)

### 9.4 IDEA 集成码云
**9.4.1 IDEA 安装码云插件**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078747777-e2844af2-9d84-4cc0-af01-8cf38d0f511d.png)  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078747781-e27a3a8b-84ac-4d2a-bc4a-fccd15157c84.png)  
Idea 链接码云和链接 GitHub 几乎一样，安装成功后，重启 Idea。  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078747806-37981ad4-9758-452e-b602-bca75282e1e7.png)  
Idea 重启以后在 Version Control 设置里面看到 Gitee，说明码云插件安装成功。  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078747802-aee8b2fd-793a-4292-8f39-c05508ab6846.png)  
然后在码云插件里面添加码云帐号，我们就可以用 Idea 连接码云了。

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078748006-66382e7f-c49a-44fe-9b3d-582a3f5fbc16.png)  
**9.4.2 IDEA 连接码云**

Idea 连接码云和连接 GitHub 几乎一样，首先在 Idea 里面创建一个工程，初始化 git 工程，然后将代码添加到暂存区，提交到本地库，这些步骤上面已经讲过，此处不再赘述。

➢ 将本地代码 push 到码云远程库  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078748130-37a39cae-4adb-4f53-aae3-2b4f4132b5ec.png)  
自定义远程库链接。  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078748161-bae9193f-b7fd-4d9c-9ecd-5a4b34b0ba86.png)  
给远程库链接定义个 name，然后再 URL 里面填入码云远程库的 HTTPS 链接即可。码云服务器在国内，用 HTTPS 链接即可，没必要用 SSH 免密链接。  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078748192-80f0d39c-173c-4bda-b0ae-15381d59c8b0.png)  
然后选择定义好的远程链接，点击 Push 即可。  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078748217-88735810-49a3-4822-a46e-d554a94d0c1d.png)  
看到提示就说明 Push 远程库成功。  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078748380-45251abe-17f2-4509-aebb-7cecd2850d05.png)  
去码云远程库查看代码。  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078748493-a8746f9c-913d-4ea6-8165-2899518fc74e.png)  
只要码云远程库链接定义好以后，对码云远程库进行 pull 和 clone 的操作和 Github 一致，此处不再赘述。

### 9.5 码云复制 GitHub 项目
码云提供了直接复制 GitHub 项目的功能，方便我们做项目的迁移和下载。  
具体操作如下：  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078748535-1703e1dc-f419-425d-aaae-b5b5291a0861.png)  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078748587-75ba8ed9-0392-4d1e-b527-54f079243fe8.png)  
如果 GitHub 项目更新了以后，在码云项目端可以手动重新同步，进行更新！  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078748642-46041656-30fa-428b-8606-0e6809ee0c3e.png)  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078748782-02126d57-6d81-4207-baec-26b385f1002c.png)  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078748863-1f63a8fe-5dff-46a3-a5ed-7d3159e94bc5.png)

## 10、自建代码托管平台-GitLab
### 10.1 GitLab 简介
GitLab 是由 GitLabInc.开发，使用 MIT 许可证的基于网络的 Git 仓库管理工具，且具有wiki 和 issue 跟踪功能。使用 Git 作为代码管理工具，并在此基础上搭建起来的 web 服务。GitLab 由乌克兰程序员 DmitriyZaporozhets 和 ValerySizov 开发，它使用 Ruby 语言写成。后来，一些部分用 Go 语言重写。截止 2018 年 5 月，该公司约有 290 名团队成员，以及 2000 多名开源贡献者。GitLab 被 IBM，Sony，JülichResearchCenter，NASA，Alibaba，Invincea，O’ReillyMedia，Leibniz-Rechenzentrum(LRZ)，CERN，SpaceX 等组织使用。

### 10.2 GitLab 官网地址
官网地址：[https://about.gitlab.com/](https://about.gitlab.com/)  
安装说明：[https://about.gitlab.com/installation/](https://about.gitlab.com/installation/)

### 10.3 GitLab 安装
**10.3.1 服务器准备**  
准备一个系统为 CentOS7 以上版本的服务器，要求内存 4G，磁盘 50G。关闭防火墙，并且配置好主机名和 IP，保证服务器可以上网。此教程使用虚拟机：主机名：gitlab-server IP 地址：192.168.6.200  
**10.3.2 安装包准备**  
Yum 在线安装 gitlab- ce 时，需要下载几百 M 的安装文件，非常耗时，所以最好提前把所需 RPM 包下载到本地，然后使用离线 rpm 的方式安装。  
下载地址：

```plain
https://packages.gitlab.com/gitlab/gitlab-ce/packages/el/7/gitlab-ce-13.10.2-ce.0.el7.x86_64.rpm
```

**10.3.3 编写安装脚本**  
安装 gitlab 步骤比较繁琐，因此我们可以参考官网编写 gitlab 的安装脚本。

```plain
[root@gitlab-server module]# vim gitlab-install.sh
sudo rpm -ivh /opt/module/gitlab-ce-13.10.2-ce.0.el7.x86_64.rpm

sudo yum install -y curl policycoreutils-python openssh-server cronie

sudo lokkit -s http -s ssh

sudo yum install -y postfix

sudo service postfix start

sudo chkconfig postfix on

curl https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.rpm.sh | sudo bash

sudo EXTERNAL_URL="http://gitlab.example.com" yum -y install gitlab-ce
```

给脚本增加执行权限  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078748926-fbe82d5a-1781-48c6-9c44-8d6ac034e1b9.png)  
然后执行该脚本，开始安装 gitlab-ce。注意一定要保证服务器可以上网。  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078748938-72c732ed-2b6c-4eb3-8300-2791a423cdeb.png)  
**10.3.4 初始化 GitLab 服务**  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078749107-6a190873-6d8a-4f97-b6f3-36185ba42799.png)  
**10.3.5 启动 GitLab 服务**  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078749195-dec83ffc-55ce-4c26-b087-ed5d2c725401.png)  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078749191-629c3694-5ed7-43da-8f23-66a3833ff75d.png)  
**10.3.6 使用浏览器访问 GitLab**  
使用主机名或者 IP地址即可访问GitLab服务。需要提前配一下windows 的hosts 文件。  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078749280-24b8e43b-0fab-4719-a7f8-e38986e1689d.png)  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078749304-df6214e7-bd05-4613-be25-29f0a28ca5f8.png)  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078749533-e24ea58d-4f95-487a-918b-5f853908d8af.png)  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078749581-403c16eb-15ab-45e1-89f1-b97ed8203141.png)  
**10.3.7 GitLab 创建远程库**  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078749705-00b54d6d-5bb4-4dc6-8416-a997a80401a6.png)  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078749620-2dd7f23e-4e2d-4a70-8672-43db2d8967e3.png)  
**10.3.8 IDEA 集成 GitLab**  
➢ 1）安装 GitLab 插件  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078749701-6435e973-41f9-47bd-9c76-833bf4d402c9.png)  
➢ 2）设置 GitLab 插件

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078749953-02e0d4a0-167a-4814-bed7-63ff5b385cae.png)  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078749965-950ecc5a-7b71-49a5-9e32-b01b1ef3d0a8.png)  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078750032-ad358c3f-ee11-414b-9b0d-04ee3e0d6dc9.png)  
➢ 3）push 本地代码到 GitLab 远程库

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078750130-157a5a5d-c599-470c-8477-e36dead1ebcd.png)  
自定义远程连接  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078750152-eb2c141d-e0ee-4a6b-bd07-18511850b83d.png)  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078750365-b9c423d6-3f13-4ec7-ac68-c78b9b1e8ae0.png)  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078750393-b600fae8-247e-4b20-b371-f53487eed449.png)  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078750428-f0bdf038-f117-4e9f-b464-a8b4ae39a80c.png)  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1762078750524-8f63c4dd-6492-4d71-91c3-4edfcfd7252e.png)

## 11、工作中正确使用git
1、 先把master主分支上的代码与远程仓库同步到最新版本。直接从远程仓库pull下来  
2、 在本地创建并切换到工作分支，创建完成后会自动切换到该分支（这时候master/work分支的内容是相同的。）  
3、 编写代码。  
4、 完成后，add commit到work分支  
5、 将主分支从远程仓库pull最新的版本下来。  
6、 将工作分支合并到主分支

+ 先切换到主分支master
+ 选中要合并的分支work，点击merge
+ 如果有冲突，则需要手动解决冲突

7、 这时候所有在work分支上做得修改就会合并到主分支上。  
8、 最后就可以push到远程仓库上了。

`关于分支切换的两个注意点`

+ 分支切换一定要先add/commit当前分支上的修改，然后如果在修改完代码后没有提交，就想切换，idea会提示是否进行smart checkou，如果你点击yes你就完完了，idea会把当代分支上的修改，保存到你要切换的另一个分支上。这样一样就乱套了。
+ 如果当前工作分支上有很多bug不想提交，那么你可以先隐藏当前工作分支上的修改，stash（隐藏），然后切换到另一个分支上，那么下次你又切换回工作分支的时候，你可以通过unstash把修改的代码重新显示出来。

