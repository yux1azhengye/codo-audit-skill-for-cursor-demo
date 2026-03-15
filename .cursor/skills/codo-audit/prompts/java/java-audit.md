# Java 技术栈 Recon 与 Sink 策略

## 技术栈识别

- 构建：`pom.xml`（Maven）、`build.gradle`（Gradle）。
- 框架：Spring Boot/Spring MVC、JAX-RS、Struts、Servlet 等。
- ORM：MyBatis、Hibernate、JPA。
- 模板：Thymeleaf、Freemarker、JSP。

## 路由与入口

- Spring：`@RequestMapping`、`@GetMapping`/`@PostMapping`、`@RestController`；注意 `@ControllerAdvice`、拦截器、`Filter`、`HandlerInterceptor` 对请求的包装与鉴权。
- 参数入口：`@RequestParam`、`@RequestBody`、`@PathVariable`、`HttpServletRequest.getParameter()` 等。

## 常见 Sink（供 Round 1 选型）

| 类型     | 典型 Sink / 模式 |
|----------|------------------|
| SQL      | `Statement`/`PreparedStatement` 拼接、MyBatis `${}`、Hibernate HQL 拼接、`JdbcTemplate` 拼接 |
| RCE/执行 | `Runtime.getRuntime().exec()`、`ProcessBuilder`、`ScriptEngine`、OGNL（Struts） |
| SSRF     | `URL.openConnection()`、`HttpClient`/`HttpURLConnection` 使用用户可控 URL、`RestTemplate` |
| 文件     | `FileInputStream`/`FileOutputStream`、`Paths.get()`、`new File()`、读写用户可控路径 |
| 反序列化 | `ObjectInputStream.readObject()`、`JSON.parseObject`（Fastjson 等）、XML 反序列化 |
| 逻辑/鉴权 | `@PreAuthorize`、`@Secured`、自定义 Filter 中的 `user_id`/`role` 校验、Service 层权限判断 |

## 逻辑漏洞上下文

- 注解与拦截器：若 API 通过注解或配置注入鉴权/过滤器，AST 可能无法直接看到“该路由经过某 Filter”。Recon 时单独提取：哪些路由标注了权限、哪些 Filter 对 path 生效，供业务逻辑专项按「路由 + Service 层」追踪。
