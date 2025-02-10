---
layout: post
author: Aki
tags: [Springboot, バグ]
---

## Table of contents

- [RESTful API](#restful-api)

- [Entity & DTO](#entity-与-dto-的区别)

---

### RESTful API

1. **资源（Resource）**：  
   资源就是系统中的数据，比如用户、商品、文章。每个资源都有唯一的 URL。  
   - 例如：`https://example.com/users/1` 表示 ID 为 1 的用户。

2. **HTTP 方法**：  
   RESTful API 通过 HTTP 请求来操作资源，不同的操作使用不同的 HTTP 方法：
   - **GET**：获取资源，比如查看某个用户的数据。
   - **POST**：创建资源，比如新增一位用户。
   - **PUT**：更新资源，比如修改用户信息。
   - **DELETE**：删除资源，比如删除某个用户。

3. **无状态**：  
   每次请求都是独立的，服务器不会记住你上次请求的状态。所有信息都必须包含在每个请求里，比如身份认证信息。

4. **返回数据格式**：  
   RESTful API 通常返回的数据格式是 **JSON**（轻量、易读），也可以是 XML，但JSON更常用。

#### 举个例子

假设我们有一个书籍管理系统：

- 获取所有书籍：  
  **GET** `https://api.example.com/books`

- 获取某本书（ID 为 1）：  
  **GET** `https://api.example.com/books/1`

- 创建一本新书：  
  **POST** `https://api.example.com/books`  
  传递数据（JSON 格式）：
  ```json
  {
    "title": "Spring Boot Guide",
    "author": "John Doe"
  }
  ```

- 更新一本书的信息：  
  **PUT** `https://api.example.com/books/1`

- 删除一本书：  
  **DELETE** `https://api.example.com/books/1`

#### Springboot `@RestController` VS `@Controller`


| 特性                   | @Controller                        | @RestController                  |
|------------------------|------------------------------------|----------------------------------|
| 视图解析                | 是（通过视图解析器返回HTML等视图）   | 否（直接返回数据）               |
| 默认响应内容            | 视图名称                           | JSON、XML、字符串等数据           |
| 是否需要 `@ResponseBody` | 需要（如果直接返回数据）            | 不需要                           |
| 典型应用场景            | 返回网页视图（如HTML）              | 构建REST API，返回JSON或其他数据格式 |

**`@RestController`** 更适合开发RESTful API，直接返回数据。而 **`@Controller`** 适合传统的网页应用，返回视图页面。

如果你主要是在做后端API开发，`@RestController` 会更方便。如果你是在开发返回HTML页面的应用，`@Controller` 会更合适。

---

### Entity 与 DTO 的区别

- **Entity（实体类）**
  - **作用**：用于数据持久化，与数据库表直接映射。
  - **特点**：
    - 带有 JPA 注解（如 `@Entity`、`@Table`、`@Id`）。
    - 包含所有数据库字段（包括敏感数据，如密码）。
    - 用于 CRUD 操作，反映真实的数据库结构。

- **DTO（数据传输对象）**
  - **作用**：用于在各层（如前后端）之间传递数据。
  - **特点**：
    - 是简单的 POJO，不包含持久化注解。
    - 只包含业务需要展示或传输的字段，避免暴露敏感信息。
    - 解耦内部数据库模型与外部接口的数据格式。

**典型使用流程**

1. **数据查询**：从数据库中查询出 Entity 对象。
2. **数据转换**：将 Entity 转换为 DTO，过滤掉不必要或敏感的字段。
3. **数据传输**：将 DTO 返回给前端或其他层。

**总结：**
- Entity 用于数据持久化，直接反映数据库表结构，适合存储和管理业务数据；而 DTO 则用于数据传输，封装前后端需要交互的数据，有助于分离业务逻辑和表现层，提升系统的灵活性和安全性。
- 在实际开发中，为了避免将内部数据库结构暴露给客户端，通常会先将 Entity 转换为 DTO，然后再返回给前端；同理，接收前端数据时也会先封装到 DTO，再转换为 Entity 进行业务处理和持久化操作。

---