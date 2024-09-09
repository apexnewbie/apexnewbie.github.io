---
layout: post
author: Aki
tags: [Spring Boot, バグ]
---

## Table of contents

- [RESTful API](#restful-api)

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

## Springboot `@RestController` VS `@Controller`


| 特性                   | @Controller                        | @RestController                  |
|------------------------|------------------------------------|----------------------------------|
| 视图解析                | 是（通过视图解析器返回HTML等视图）   | 否（直接返回数据）               |
| 默认响应内容            | 视图名称                           | JSON、XML、字符串等数据           |
| 是否需要 `@ResponseBody` | 需要（如果直接返回数据）            | 不需要                           |
| 典型应用场景            | 返回网页视图（如HTML）              | 构建REST API，返回JSON或其他数据格式 |

**`@RestController`** 更适合开发RESTful API，直接返回数据。而 **`@Controller`** 适合传统的网页应用，返回视图页面。

如果你主要是在做后端API开发，`@RestController` 会更方便。如果你是在开发返回HTML页面的应用，`@Controller` 会更合适。