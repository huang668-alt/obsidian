
以下是 **MVC 执行流程** 在两种不同架构下的详细对比：

---

### 一、MVC 核心概念回顾

MVC = **Model（模型） + View（视图） + Controller（控制器）**

- **Model**：数据和业务逻辑
- **View**：用户界面展示
- **Controller**：接收请求、协调 Model 和 View

---

### 二、情况一：传统视图流程（Server-Side Rendering MVC）

这是经典的 **MVC 模式**，前后端不分离，服务端直接渲染 HTML 返回给浏览器。

![[Pasted image 20260603203707.png]]



---

### 三、情况二：前后端分离流程（API 驱动 MVC）

这是现代主流方式，后端提供 **RESTful API**，前端独立负责 View 层（使用 Vue、React、Angular 等）。

![[Pasted image 20260603203844.png]]

---

