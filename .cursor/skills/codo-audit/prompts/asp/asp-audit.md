# ASP / ASP.NET 技术栈 Recon 与 Sink 策略

## 技术栈识别

- 项目：`*.csproj`、`*.sln`、`web.config`；.NET Framework 与 .NET Core/5+ 区分。
- 框架：ASP.NET MVC、ASP.NET Core MVC、Web API、Razor Pages、经典 ASP（VBScript，遗留）。
- ORM：Entity Framework、Dapper、ADO.NET。
- 模板：Razor、aspx（Web Forms）。

## 路由与入口

- ASP.NET Core：`[HttpGet]`/`[HttpPost]`、`ControllerBase`、`FromQuery`/`FromBody`/`FromRoute`、中间件管道。
- 经典 ASP：`Request.QueryString`、`Request.Form`、`Request()`。
- 参数入口：模型绑定、`Request.Form`、`Request.QueryString`、Request 头与 Cookie。

## 常见 Sink（供 Round 1 选型）

| 类型     | 典型 Sink / 模式 |
|----------|------------------|
| SQL      | `SqlCommand` 拼接、EF `FromSqlRaw`/`ExecuteSqlRaw` 拼接、Dapper 拼接 SQL、字符串拼进 SQL |
| RCE/执行 | `Process.Start`、`ProcessStartInfo`、`Assembly.Load`、反序列化 `BinaryFormatter`/`ObjectStateFormatter`、ViewState 反序列化 |
| SSRF     | `HttpClient`、`WebRequest`/`HttpWebRequest`、用户可控 URL 传入 |
| 文件     | `File.ReadAllText`/`WriteAllText`、`Path.Combine` 未校验、`FileStream`、用户可控路径 |
| 反序列化 | `BinaryFormatter.Deserialize`、`ObjectStateFormatter`、ViewState、`XmlSerializer`、JSON 反序列化到可写类型 |
| 逻辑/鉴权 | `[Authorize]`、`[AllowAnonymous]`、ClaimsPrincipal、角色/策略、手动 User.Id 与资源归属校验 |

## 逻辑漏洞上下文

- 授权：`[Authorize(Roles=...)]`、策略、Resource-based 授权；标注哪些 Controller/Action 需权限；接收 `userId`、`id`、`role` 等参数时在 Service 层是否用于鉴权、是否可 IDOR。
- 经典 ASP：若存在遗留 aspx/vbs，关注 `Request`、`Execute`、文件操作与数据库拼接。
