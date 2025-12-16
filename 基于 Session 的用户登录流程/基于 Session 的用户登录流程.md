# 基于 Session 的用户登录流程

本文从**客户端与服务器的真实交互过程**出发，详细说明基于 Session 的用户登录机制，并澄清 Session 创建时机、Cookie 作用以及会话状态的维护方式。

------

## 一、Session 与 Cookie 的基本职责

在基于 Session 的登录机制中：

- **Session**：
   存储在服务器端，用于保存用户的登录状态、身份信息等跨请求数据。
- **Cookie**：
   存储在客户端浏览器中，用于保存 SessionId，并在后续请求中自动携带给服务器。

二者通过 **SessionId** 建立关联。

------

## 二、登录流程的完整交互过程

### 1）Session 的创建（会话初始化）

当前端向服务器发起请求时，服务器**并不会在建立连接或首次接收请求时立即创建 Session**。

只有在后端代码或框架 **显式或隐式调用 `request.getSession()` 方法**时，服务器才会：

- 创建一个 `HttpSession` 对象
- 生成一个全局唯一的 **SessionId**
- 在服务器端保存该 Session 的状态

随后，服务器会在响应中通过 `Set-Cookie` 响应头将该 SessionId 返回给客户端。

> ⚠️ 注意：
>  Session 是**按需（懒加载）创建的**，是否创建完全取决于后端是否需要会话状态。

------

### 2）登录成功，更新会话信息

当用户在前端输入正确的账号和密码并提交到后端，服务器完成身份校验后，会执行以下操作：

- 获取当前请求关联的 Session（如不存在则创建）
- 将用户的登录信息（如用户 ID、用户名、角色、权限等）保存到 Session 中
- 确保 SessionId 通过 `Set-Cookie` 返回给客户端（如果是新创建的 Session）

此时，Session 中已包含该用户的身份状态。

<img src="基于 Session 的用户登录流程.assets/Set-Cookie.png" alt="Set-Cookie" style="zoom: 67%;" />

------

### 3）前端保存 Cookie

浏览器接收到服务器响应后：

- 自动解析 `Set-Cookie` 响应头
- 将 SessionId 保存到浏览器的 Cookie 中
- 并将该 Cookie 与当前域名绑定

该过程由浏览器自动完成，前端代码通常无需手动处理。

<img src="基于 Session 的用户登录流程.assets/本地cookie.png" alt="本地cookie" style="zoom: 67%;" />

------

### 4）携带 Cookie 的后续请求

在后续访问同一域名的接口时，浏览器会自动在请求头中附带之前保存的 Cookie，例如：

```
Cookie: JSESSIONID=xxxxxx
```

客户端无需感知 Session 的存在，Cookie 会随请求自动发送。

<img src="基于 Session 的用户登录流程.assets/Cookie.png" alt="Cookie" style="zoom:67%;" />

------

### 5）后端验证会话

服务器接收到请求后：

- 从请求头中读取 SessionId
- 根据 SessionId 查找服务器端对应的 Session 数据
- 将找到的 Session 绑定到当前请求上下文

此过程通常由 Web 容器（如 Tomcat）自动完成。

------

### 6）获取会话中存储的用户信息

在业务代码中，后端可以通过当前 Session：

- 获取之前存储的用户信息（如登录名、用户 ID、权限等）
- 判断用户是否已登录
- 执行相应的业务逻辑或权限校验

从而实现跨请求的用户身份识别与状态保持。

------

## 三、关键总结

- Session **不会在连接建立时自动创建**，而是在后端调用 `getSession()` 时才创建
- Cookie 只是 SessionId 的客户端载体，真正的用户数据始终存储在服务器端
- Session 绑定的是 **浏览器实例**，而不是用户本身
- 基于 Session 的登录机制本质上是通过 **SessionId + Cookie** 实现的状态保持

------

## 四、补充说明

在实际项目中，许多框架（如 Spring Security）可能会在过滤器阶段隐式创建 Session，这也是“第一次访问就存在 Session”这一常见误解的来源之一。