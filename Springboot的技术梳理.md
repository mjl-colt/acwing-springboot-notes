# Springboot的技术梳理

# 1.spring boot层次

我们先将springboot分为四个或者说是三个层次。命名为**Controller 层**、**Service 层**、**Mapper 层** （或者 命名为**Dao 层**）和 **Pojo 层**（或者 命名为**Entity 层**）。为什么说三层或者是四层你？因为pojo层是个实体类。**Controller** 调用 **Service 接口**，**Service 实现类** 调用 **Mapper 接口**，**Mapper 接口** 通过框架生成实现类操作数据库。**Entity 类** 是数据的载体，用于存储和传输数据

**第一，每个层都是干什么的**

- `pojo`层：将数据库中的表对应成`Java`中的`Class`

- `mapper`层（也叫`Dao`层）：将`pojo`层的`class`中的操作（CRUD），映射成`sql`语句

  Create => insert

  Retrieve=> select

- `service`层：写具体的业务逻辑，组合使用`mapper`中的操作

- `controller`层：负责请求转发，接受前端页面过来的参数，传给相应`Service`处理，接到返回值，再传给页面

**第二，每个层之间是怎么调用的**

1.**Controller 层** 调用 **Service 层接口**，处理请求。调度service，接受前端信息。

2.**Service 层** 调用 **Mapper 层接口**，访问数据库。可以用多个mapper，实现业务。

3.**Mapper 层** 由框架（如 MyBatis）自动生成代理实现，执行 SQL 操作。将class中的增删改查操作化成sql语句

4.**Pojo 层** 用于接收数据库查询结果或传递数据。将数据库中的table转成class类。

**第三，每个层都是写成类还是接口**

**Controller 层**：负责请求和响应（通常是类）。

**Service 层**：处理业务逻辑（接口 + 实现类），其中接口定义业务逻辑，类实现接口。

**Mapper 层**：操作数据库（通常是接口，MyBaitis框架自动实现）。

**Pojo 层 / Entity 层**：数据模型（通常是类）。

# 2.关于各种注解的说明

## **`@Autowired` **

`@Autowired` 注解是用来实现 **依赖注入** 的工具，也就是说，它可以让你在一个类中自动使用另一个类，而不需要手动创建对象。

在普通 Java 代码中，如果一个类需要另一个类，我们通常会用 `new` 来手动创建实例：

```java
public class UserController {
    private UserService userService = new UserService();
}
```

#### **在类中注入**

如果一个 `Controller` 类需要使用 `Service` 层的功能，直接在类中加 `@Autowired` 注解即可。

```java
@RestController
public class UserController {

    @Autowired // 自动注入 UserService
    private UserService userService;

    @GetMapping("/users/{id}")
    public User getUser(@PathVariable Long id) {
        return userService.getUserById(id); // 调用 userService 的方法
    }
}
```

**解释**：

- `UserController` 类中需要 `UserService`。
- 使用 `@Autowired`，Spring 会自动找到 `UserService` 的实现，并注入到 `userService` 中。
- 你不需要手动写 `new UserService()`。

#### **在构造器中注入**

你也可以通过构造器方式进行注入，这种方式推荐用于需要**不可变依赖**的情况。

**示例**：

```java
@RestController
public class UserController {

    private final UserService userService;

    // 构造器注入
    @Autowired
    public UserController(UserService userService) {
        this.userService = userService;
    }

    @GetMapping("/users/{id}")
    public User getUser(@PathVariable Long id) {
        return userService.getUserById(id);
    }
}
```

**好处**：

- 如果有多个需要注入的依赖，构造器方式更加清晰。
- 构造器注入可以保证注入的依赖在对象创建时就完成初始化。
- 使用 `private final` 修饰依赖，表明这些依赖在创建后不会再被修改（不可变）。

## **`@ResponseBody 和 @RestController`**

 `@RestController`（相当于 `@Controller + @ResponseBody`）。

如果返回值是对象或集合，`@ResponseBody` 会将其序列化为 JSON 格式返回给客户端。如果返回值是字符串，则直接返回字符串作为响应体。

#### **1. 返回 JSON 示例**

```java
@RestController
public class UserController {
    @GetMapping("/user")
    public User getUser() {
        return new User(1, "Alice");
    }
}
```

- 返回结果：

  ```
  {
      "id": 1,
      "name": "Alice"
  }
  ```

#### **2. 返回字符串示例**

```java
@RestController
public class HelloController {
    @GetMapping("/hello")
    public String sayHello() {
        return "Hello, World!";
    }
}
```

- 返回结果：

  ```
  Hello, World!
  ```

#### **3. 使用 `@Controller` + `@ResponseBody`**

如果你只标注了 `@Controller`，可以手动为方法添加 `@ResponseBody`：

```java
@Controller
public class TestController {
    @GetMapping("/test")
    @ResponseBody
    public String testResponse() {
        return "This is a test";
    }
}
```

如果没有 `@ResponseBody` 或 `@RestController`，Spring 会默认将返回值当成视图名，并尝试去解析它。会尝试寻找 `index.html` 或 `index.jsp` 文件作为视图。

例如：

```java
@Controller
public String home() {
    return "index";
}
```

## **`@RequestMapping` **

@RequestMapping说白了就是定位到页面的链接。我们需要了解一下其中的参数。看例子即可。

### **1. `value` **

- **作用**：指定请求的 URL 路径。

- **类型**：`String[]`，可以指定一个或多个路径。

- 示例：

  ```java
  @RequestMapping(value = "/user")
  public String getUser() {
      return "user";
  }
  ```

### **2. `method`**

- **作用**：限定请求方法（如 GET、POST、PUT、DELETE 等）。

- **类型**：`RequestMethod[]`。

- 示例：

  ```java
  @RequestMapping(value = "/user", method = RequestMethod.GET)
  public String getUser() {
      return "user";
  }
  ```

从 Spring 4.3 开始，推荐使用以下更具体的注解代替 `@RequestMapping`：

- **`@GetMapping`**：处理 GET 请求。
- **`@PostMapping`**：处理 POST 请求。
- **`@PutMapping`**：处理 PUT 请求。
- **`@DeleteMapping`**：处理 DELETE 请求。
- **`@PatchMapping`**：处理 PATCH 请求。

### **3. `params`**

- **作用**：限定请求中必须包含某些参数，或排除某些参数。

- **类型**：`String[]`，支持逻辑运算（如 `param1=value1`）。

- **示例**：

  ```java
  @RequestMapping(value = "/user", params = "id")
  public String getUserById() {
      return "userById";
  }
  ```

- **逻辑示例**：

  - `params = "id"`：必须包含 `id` 参数。说白了，需要请求路径中有id,例如：`http://localhost:8080/user?id`，`params = "id"`：只要有 `id` 参数即可，值无所谓。
  - `params = "!id"`：参数 `id` 不允许出现。
  - `params = "id = 8"`：参数必须为 `id = 8`。请求路径：`http://localhost:8080/user?id=8`

## **`@Service 和 @Controller 和 @Mapper 和 @Component` **

#### **1 `@Service`**

- 标记服务层的类。
- Spring 会扫描带有 `@Service` 的类，并将其实例化为 Bean。

#### **2 `@Controller`**

- 标记控制器层的类。
- 通常和 Web 应用中的请求映射（`@RequestMapping`）一起使用，用于处理 HTTP 请求。

#### **3 `@Mapper`**

- 通常配合 MyBatis 使用。
- `@Mapper` 是 MyBatis 提供的注解，用于标记数据访问层接口。
- Spring 会自动将这些接口注册为 Bean，并生成动态代理对象。

#### **4 `@Component`**

- 通用的组件注解，任何不属于 `@Service`、`@Controller`、`@Repository` 的类都可以用它标记。
- 被标记的类会被注册为 Bean。

### **运行流程**

1. **启动项目**：
   - 启动 `BackendApplication`。
2. **发送 HTTP 请求**：
   - 访问 `http://localhost:8080/user/1`。
3. **Spring 工作流程**：
   1. 控制器处理请求：
      - `UserController` 通过 `userService.getUserById(1)` 调用服务层。
   2. 服务层调用 Mapper：
      - `UserService` 调用 `UserMapper.findUserById(1)`。
   3. 返回结果：
      - `Mapper` 返回用户对象，最终控制器返回 JSON 响应。

------

### **容器管理的 Bean**

- Spring 会创建这些类的实例（通过反射）。

  将实例注册到 IOC 容器中，Bean 的名称默认为类名首字母小写。

- Spring 会自动将以下类作为 Bean 注入到 IOC 容器：
  1. `@Service` 标记的 `UserService`。
  2. `@Controller` 标记的 `UserController`。
  3. `@Mapper` 标记的 `UserMapper`。
- 这些类会被自动注入到需要的地方（如通过 `@Autowired`）。

# 3.一些小问题。

**问题：**

第一，public interface UserMapper extends BaseMapper<User> { }这个UserMapper不是一个接口吗？怎么可能直接写userMapper.insert(user);这句话呢？

第二，结合上面说的`@Autowired` 注入，在代码里@Autowired UserMapper userMapper;为什么能自动注入一个接口？

**解决：**

Spring 和 MyBatis 配合使用时，**Spring 会自动为接口生成实现类的代理对象**，并将这个代理对象注入到应用的上下文中。

### **总结过程**

1. **动态代理的生成**：

   - `UserMapper` 是接口，无法直接调用。
   - Spring 在运行时生成一个代理类，代理类实现了 `UserMapper` 接口。
   - 代理类通过 MyBatis 调用底层数据库操作。

2. **依赖注入**：

   - Spring 将代理对象注入到你的代码中（比如通过 `@Autowired`）。
   - 因此，`userMapper` 实际上指向的是代理对象，而不是接口本身。

3. **调用代理类方法**：

   - 调用 `userMapper.insert(user)` 时，代理类会根据 `BaseMapper` 的实现逻辑，生成并执行对应的 SQL 语句。

   其实说白了都是对代理类进行操作，它是一个运行时生成的类，动态生成的类，通常是由JDK 动态代理实现。

**问题：**

第三，QueryWrapper<User> queryWrapper = new QueryWrapper<>();怎么结合 MyBatis-Plus使用。

**解决：**

**`QueryWrapper<User>`**是一个泛型类，`QueryWrapper<T>` 是 MyBatis-Plus 提供的条件构造器，用于构建数据库查询的条件。泛型 `<User>` 表示这个条件构造器是针对 `User` 实体类的，用于查询 `User` 对应的数据表。

`QueryWrapper` 是 MyBatis-Plus 中用来构造动态 SQL 的工具，主要作用是：

1. 用代码的方式代替传统的 XML 或字符串拼接来构建 SQL 条件。
2. 支持链式调用，简化查询条件的书写。
3. 自动生成语义化的 SQL 语句。

说白了QueryWrapper就是集成了SQL语句的代码表示方式。例子如下，这两个是等效的：

```java
QueryWrapper<User> queryWrapper = new QueryWrapper<>();
queryWrapper.eq("username", "Alice")  // WHERE username = 'Alice'
            .gt("age", 18)           // AND age > 18
            .like("email", "@gmail.com"); // AND email LIKE '%@gmail.com%'

```

```sql
SELECT * FROM user WHERE username = 'Alice' AND age > 18 AND email LIKE '%@gmail.com%';
```

![d6a158a40f3b580eb82e5aa46158080](C:\Users\30816\OneDrive\文档\WeChat Files\wxid_5p4nkpnvp39122\FileStorage\Temp\d6a158a40f3b580eb82e5aa46158080.png)

通过 `eq`, `like`, `gt` 等方法添加条件，再传递给 MyBatis-Plus 的 `Mapper` 方法（如 `selectList` 或 `selectOne`）

# 4.**Session** 和 **JWT**的应用（用于验证的）session比jwt老一点，现在都是用jwt。

### **Session 的示例流程**

1. **用户登录：**

   - 用户输入用户名和密码：
     用户名：`us`
     密码：`123`
   - 客户端将这些数据发送给服务器进行验证。

2. **服务器验证：**

   - 服务器检查用户名和密码是否匹配：

     查询数据库：

     ```
     用户名：us  
     密码：123  -> 验证成功
     ```

   - 验证通过后，服务器生成一个 **Session ID**：`147852369`。

3. **存储 Session：**

   - 服务器将 Session ID与用户信息绑定，存储在服务器的 Session 存储中，例如：

     ```
     Session ID: 147852369  
     用户信息：us  
     过期时间：30分钟
     ```

4. **返回 Session ID：**

   - 服务器将 Session ID返回给客户端，并存储在客户端的 Cookie 中：

     ```
     Set-Cookie: SessionID=147852369; Expires=30min
     ```

5. **后续请求：**

   - 客户端每次发起请求时，自动将 `SessionID=147852369` 放在请求头中发送给服务器。
   - 服务器根据 `Session ID` 找到对应的用户信息完成验证。

### **JWT 的示例流程**

1. **用户登录：**

   - 用户输入用户名和密码：
     用户名：`us`
     密码：`123`
   - 客户端将这些数据发送给服务器进行验证。

2. **服务器验证：**

   - 服务器检查用户名和密码是否匹配：

     查询数据库：

     ```
     用户名：us  
     密码：123  -> 验证成功
     ```

3. **生成 JWT：**

   - 服务器生成一个 JWT，包含用户信息和签名：

     ```
     Header: { "alg": "HS256", "typ": "JWT" }
     Payload: { "username": "us", "role": "user", "exp": 1680000000 }
     Signature: HMACSHA256(base64UrlEncode(header) + "." + base64UrlEncode(payload), secret)
     ```

     - 示例密钥（Secret）：`my_secret_key`

     - 生成的 JWT：

       ```
       复制代码
       eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6InVzIiwicm9sZSI6InVzZXIiLCJleHAiOjE2ODAwMDAwMDB9.abc123def456ghi789jkl
       ```

   我们详细说一下这个JWT的生成，JWT 由三个部分组成，用 `.` 分隔：Header.Payload.Signature，其中意思分别是**Header（头部）**，**Payload（负载）**，**Signature（签名）**。

    **Header（头部）**

   Header 是一个 JSON 对象，用来描述 Token 的元信息，很多jwt头部都是一样的。

   ```json
   {
     "alg": "HS256", // 签名算法 HMAC-SHA256(这是Header和Payload都写完之后，用来和密钥一起进行加密的)
     "typ": "JWT"    // 表明这是一个 JWT
   }
   ```

   这个 JSON 对象会被进行 **Base64Url 编码**（与 Base64 类似，但对 URL 友好），编码结果：

   ```
   eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9
   ```

   ### **Payload（负载）**

   Payload 是实际存储信息的地方，也是一个 JSON 对象，例如：

   ```json
   {
     "username": "us",     // 用户名
     "role": "user",       // 用户角色
     "exp": 1680000000     // 过期时间（Unix 时间戳）
   }
   ```

   将此 JSON 对象同样进行 **Base64Url 编码**，结果：

   ```
   eyJ1c2VybmFtZSI6InVzIiwicm9sZSI6InVzZXIiLCJleHAiOjE2ODAwMDAwMDB9
   ```

   ### **Signature（签名）**

   签名是用来验证数据完整性和真实性的，防止数据被篡改。生成签名的步骤如下：

   1. 将编码后的 Header 和 Payload 拼接，用**.**分隔：

      ```
      eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6InVzIiwicm9sZSI6InVzZXIiLCJleHAiOjE2ODAwMDAwMDB9
      ```

   2. 使用密钥（Secret）对拼接字符串进行 HMAC-SHA256 签名。假设密钥是：my_secret_key

      HMAC 签名过程：（这是需要算法和密钥同时协作才能搞出来的）

      ```
      Signature = HMACSHA256(
          base64UrlEncode(Header) + "." + base64UrlEncode(Payload), 
          secret
      )
      ```

   3. 生成的签名进行 Base64Url 编码，例如：

      ```
      abc123def456ghi789jkl
      ```

   最终的完整 JWT：

   ```
   eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6InVzIiwicm9sZSI6InVzZXIiLCJleHAiOjE2ODAwMDAwMDB9.abc123def456ghi789jkl
   ```

4. **返回 JWT：**

   - 服务器将生成的JWT返回给客户端：

     ```
     Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6InVzIiwicm9sZSI6InVzZXIiLCJleHAiOjE2ODAwMDAwMDB9.abc123def456ghi789jkl
     ```

5. **后续请求和服务器验证 JWT：**

   客户端每次发起请求时，将JWT放在请求头中：

   - ```
     Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6InVzIiwicm9sZSI6InVzZXIiLCJleHAiOjE2ODAwMDAwMDB9.abc123def456ghi789jkl
     ```

   服务器收到 JWT 后，通过以下步骤验证：

   1. 提取 JWT 中的 Header 和 Payload，重新计算签名：

      ```
      Signature = HMACSHA256(
          base64UrlEncode(Header) + "." + base64UrlEncode(Payload), 
          secret
      )
      ```

   2. 比较计算出来的签名和 JWT 中的签名是否一致：

      - **一致：** 说明数据未被篡改，验证通过。
      - **不一致：** 数据可能被篡改，拒绝处理。

   3. 检查 Payload 的过期时间：

      ```
      当前时间 < exp（过期时间） -> Token 有效
      当前时间 ≥ exp -> Token 已过期
      ```

   **Session**：所有用户信息都在服务器端存储，客户端只持有一个 Session ID。

   **JWT**：用户信息直接存储在 JWT 中的Payload 部分（客户端持有），服务器只需要验证 JWT 签名即可使用这些信息。

我们后续都使用jwt，在跨域操作和分布式效果更好。

# 5.在Security中取用户

首先说jwt和Spring Security

**JWT 是一种“身份凭证”机制，Spring Security 是一个“安全框架”。**

它们俩不是一回事，但可以**配合使用**，就像你用身份证（JWT）进门，门卫（Spring Security）负责验证你的身份，决定你能不能进去、能不能做事。

Spring Security 是一个框架，用来做：

- 用户认证（你是谁）
- 用户授权（你能做什么）
- 会话管理、安全过滤器、权限控制等

但是 Spring Security 并不规定你用哪种方式认证 —— 你可以用表单登录、OAuth、LDAP、JWT……都可以。

![4](C:\Users\30816\OneDrive\桌面\Springoot项目笔记\image\4.png)

```java
// 从安全上下文中获取“当前登录的用户信息” 
UsernamePasswordAuthenticationToken authenticationToken =
        (UsernamePasswordAuthenticationToken) SecurityContextHolder.getContext().getAuthentication();

// 从这个 token 中拿到我们的登录用户对象（实现了 UserDetails 接口）
UserDetailsImpl loginUser = (UserDetailsImpl) authenticationToken.getPrincipal();

// 从 loginUser 里拿出我们自己系统定义的 User 对象
User user = loginUser.getUser();
```

登录用户对象和User的区别：假设你公司里有一个员工小明：

- 公司 IT 系统里有一张员工表（ID、姓名、年龄、职位）—— 这就是 `User`
- 安保系统为了识别员工出入权限，会再生成一个“登录信息对象”，包括小明的账号、权限列表、登录时间等 —— 这就是 `LoginUser` 或 `UserDetailsImpl`

下面这个是登录阶段的做法：

![5](C:\Users\30816\OneDrive\桌面\Springoot项目笔记\image\5.png)

![6](C:\Users\30816\OneDrive\桌面\Springoot项目笔记\image\6.png)

# 6.service，serviceimp,controller怎么互相调用

来一个例子。

### ① Spring 启动时会做什么？

当你启动 Spring Boot 应用时，Spring 会扫描所有带有注解的类，比如：

```java
@Service
public class UpdateServiceImpl implements UpdateService { ... }
```

看到这个 `@Service` 注解，Spring 会自动把这个类注册为一个 **Bean**，并放入 Spring 容器（ApplicationContext）中。

> ✅ Bean 是 Spring 容器托管的对象，也就是我们不自己 `new`，而让 Spring 自动创建、管理生命周期。

------

### ② 为什么接口能注入实现类？

当 Spring 看到你这样写：

```java
@Autowired
private UpdateService updateService;
```

它会在容器里查找有没有实现了 `UpdateService` 接口的类。

发现你定义了：

```java
@Service
public class UpdateServiceImpl implements UpdateService { ... }
```

就会把这个类创建一个对象，然后注入进来。

```java
你定义了：
    interface UpdateService {...}
    class UpdateServiceImpl implements UpdateService {...}

Spring 启动时：
    ┌──────────────┐
    │ UpdateController (使用方)
    └────┬─────────┘
         │
         ▼
@Autowired UpdateService updateService

Spring 容器中查找 Bean：
    ┌───────────────────────────────┐
    │ 找到：UpdateServiceImpl 实现类 │
    └────────┬──────────────────────┘
             │
             ▼
    注入对象：new UpdateServiceImpl()（由Spring管理）

```

`@Autowired` 注入的是接口，Spring 会帮你找唯一对应的实现类，自动创建并注入，前提是这个类被加了 `@Service` 或 `@Component` 注解，已注册为 Bean。

同样的道理,下面这个注入时候的代理类，**这个实现类是 MyBatis-plus 帮你自动生成的** 。

```java
@Autowired
private BotMapper botMapper;
```

```java
@Mapper
public interface BotMapper extends BaseMapper<Bot> {
    // 你甚至都可以不写方法，MyBatis-Plus 帮你自动生成了很多方法
}
```

Spring + MyBatis 会：

1. 扫描所有 `@Mapper` 接口；
2. 为每个 Mapper 接口生成一个 **代理类**（其实是动态生成的实现类）；
3. 注册到 Spring 容器；
4. 你 `@Autowired` 注入的时候，它就注入这个代理对象。

# 7.websocket的知识点

websocket是用来维护匹配系统的服务端和客户端的连接，**建立持久连接**的通信协议，跟传统的 HTTP 不同。它可以全双工，连接持续存在。在用户开始匹配的时候，每个client向后端发送一个请求，就会在后端开辟一个线程，创建并维护一个websocket连接（实际上就是new一个WebSocketServer类的实例）。**WebSocket 就是为了让服务端和客户端之间能“实时保持连接”，从而实时推送匹配结果、状态等关键事件的！**

`WebSocketServer` 类是由 **WebSocket 容器** 管理的，而不是由 Spring 容器管理的。所以没法直接@Autowired注入，要换成别的方式，如下。

```java
private static UserMapper userMapper;

@Autowired
public void setUserMapper(UserMapper userMapper){
    WebSocketServer.userMapper = userMapper;
}
```

这个 `Session`**表示一个客户端与服务端的 WebSocket 会话**。每个用户连接一次，就会生成一个独立的 `Session`。

**WebSocket 的 `Session` 是在客户端与服务器成功建立连接时，由 WebSocket 容器自动创建的。**

```
前端连接：
ws://localhost:8080/websocket/3  (token = "3")

↓ onOpen()
1. session 记录连接
2. 用 token 找到用户
3. 把 userId -> WebSocketServer 实例 存到 map

↓ onMessage()
4. 接收消息（你可以实现逻辑）

↓ sendMessage()
5. 向该连接的客户端发消息

↓ onClose()
6. 客户端断开连接，移除映射
```

# 8.spring security的分析。

首先SecurityConfig是进行Security的配置，里面有一行代码，对哪些路径进行赦免，不安全检测。

```java
.antMatchers("/api/user/account/token/", "/api/user/account/register/").permitAll()
```

这些接口 **允许匿名访问**，不会被 Spring Security 拦截器处理。说白了这两个链接，就是不验证就可以通过。

##  1.分析没被拦截的链接(只有注册和登录页面)

### 1.1 如果是登录

如果登录接口也走 Spring Security 的认证拦截流程，就会变成**先要登录才能登录**，造成死循环！所以它放开。

登录接口 `/api/user/account/token/`：

- 前端发送用户名、密码；
- 后端 `LoginController` → `LoginServiceImpl.getToken()`；
- 手动调用 `authenticationManager.authenticate()` 验证；
- 验证通过，生成 JWT 返回给前端。

```java
//你手动创建了一个 Authentication 对象，里面装的是用户名和明文密码
UsernamePasswordAuthenticationToken authenticationToken = new UsernamePasswordAuthenticationToken(username, password);
//调用 authenticationManager.authenticate()
Authentication authenticate = authenticationManager.authenticate(authenticationToken);
```

- 它的作用是 **手动调用 Spring Security 的“用户名+密码验证逻辑”**。
- 验证过程会自动走你配置的 `UserDetailsService`，你其实已经实现了它：`UserDetailsServiceImpl`，它会查数据库里的用户信息并比对密码。即第二行代码

### 1.2 如果是注册

注册逻辑只是把用户信息写入数据库，不需要验证密码对不对，因为前端传来的密码是要新存进去的，没得比。所以注册接口直接用：

```java
userMapper.insert(user);
```

就行了。

## 2.分析被拦截的链接

```java
.anyRequest().authenticated() // 所有未写 permitAll 的接口，默认都需要登录
```

对战开始 `/pk/start/game/`：

- 前端带 JWT 调用接口；

- `JwtAuthenticationTokenFilter` 拦截器提取 Token → 解析用户 ID；

- 查询数据库，拿到 User → 封装成 `Authentication` 对象放入上下文；

- 然后放行给 Controller 正常执行业务。

  

**SecurityConfig 的代码不会在每次 HTTP 请求时运行**，它只在应用启动时配置好 Spring Security 的过滤器链，之后过滤器链会在每次请求时处理。**JwtAuthenticationTokenFilter 是一个过滤器**，它在 **每次 HTTP 请求**时被调用，执行具体的拦截逻辑（如验证 JWT 令牌、设置认证信息）。

注意，无论网页是否被拦截，都会通过JwtAuthenticationTokenFilter这个过滤器。

如果是未被拦截的链接,因为SecurityConfig 中写的是**permitAll() 路径**，所以，JwtAuthenticationTokenFilter 会检查是否有 JWT 令牌，如果没有或格式不对，直接放行（filterChain.doFilter）。代码如下：

```java
if (!StringUtils.hasText(token) || !token.startsWith("Bearer ")) {
            filterChain.doFilter(request, response);
            return;
}
```

如果是**web.ignoring() 路径**：这些路径完全绕过 Spring Security 的过滤器链，因此不会经过 JwtAuthenticationTokenFilter。

但我们的SecurityConfig中写的是permitAll()。所以都会经过。

# 9.UserDetailsImpl和UserDetailsServiceImpl的理解



```pgsql
           ┌────────────────────┐
           │  LoginServiceImpl  │  ⬅️ 你自己调用登录逻辑
           └────────────────────┘
                      │
                      ▼
  authenticationManager.authenticate(authenticationToken)
                      │
                      ▼
     ┌────────────────────────────────────────────┐
     │       Spring Security 的 ProviderManager     │  ⬅️ 内部使用 UserDetailsService
     └────────────────────────────────────────────┘
                      │
                      ▼
       ┌────────────────────────────────┐
       │ UserDetailsServiceImpl（你写的）│  ⬅️ 按用户名查数据库
       └────────────────────────────────┘
                      │
                      ▼
       ┌─────────────────────────────┐
       │  UserDetailsImpl（你写的）   │  ⬅️ 包装数据库查到的 User 对象
       └─────────────────────────────┘
       
```

## `UserDetailsImpl`

这是你实现的 **Spring Security 的用户封装类**，实现了 `UserDetails` 接口。

👉 它的作用是：把你自己定义的 `User` 类（一般是数据库对象）封装成 Spring Security 认识的对象。

## `UserDetailsServiceImpl` 

这是你实现的 **Spring Security 的用户加载器**，实现了 `UserDetailsService` 接口。

👉 它的作用是：**根据用户名从数据库查出用户，并返回 `UserDetailsImpl`。**

| 类名                     | 作用描述                                           |
| ------------------------ | -------------------------------------------------- |
| `UserDetailsImpl`        | 告诉 Security：我的用户包含什么信息（用户名/密码） |
| `UserDetailsServiceImpl` | 告诉 Security：我的用户应该怎么从数据库里查出来    |

🧠 为什么要这么设计？ 

因为 Spring Security 要支持各种不同的数据库、用户结构，它不知道你用什么 User 类，也不知道你怎么查数据库，所以要你自己实现：

 UserDetailsImpl：告诉它你的 User 有哪些字段 

UserDetailsServiceImpl：告诉它怎么查你的 User 

🧩 最后一句话总结： 

UserDetailsServiceImpl 负责查用户，UserDetailsImpl 负责描述用户，这两个类构成了 Spring Security 登录认证的定制入口。我不懂，给我个例子。

# 10.前后端交互的一个问题

**后端的数据不会直接到组件，而是通常先通过 Vuex 来统一处理和管理**。组件不直接处理后端响应，而是把登录逻辑**交给 Vuex 的 action**，由 Vuex 统一发送请求、处理响应、更新状态，组件只通过回调拿到“结果”。

比如，在LoginServiceImpl中：

```java
Map<String, String> map = new HashMap<>();
//注意传给前端1处
map.put("error_message","success");
//注意传给前端2处
map.put("token",jwt);
return map;
```

此时返回是一个map给前端，前端肯定要处理，这是给vuex中的action来处理的。

```js
login(context, data){
            $.ajax({
                url:"https://app3154.acapp.acwing.com.cn/api/user/account/token/",
                type:"post",
                data:{
                    username:data.username,
                    password:data.password,
                },
                success(resp){
                    //注意接受后端1处
                    if(resp.error_message === "success"){
                        localStorage.setItem("jwt_token", resp.token);
                        context.commit("updateToken",resp.token);
                        data.success(resp);
                    }else{
                        data.error(resp);
                    }
                },
                error(resp){
                    data.error(resp);
                }
            })
        },
```

