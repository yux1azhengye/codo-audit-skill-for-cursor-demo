# Go 技术栈 Recon 与 Sink 策略

## 技术栈识别

- 模块：`go.mod`。
- 框架：Gin、Echo、Fiber、标准库 `net/http`。
- ORM/DB：`database/sql`、GORM、sqlx。

## 路由与入口

- Gin：`router.GET/POST(...)`、`c.Param()`、`c.Query()`、`c.PostForm()`、`c.Bind()`。
- 中间件：`Use()` 注册的鉴权、日志等；需在 Recon 中列出哪些路由组或路径使用了鉴权中间件。

## 常见 Sink（供 Round 1 选型）

| 类型     | 典型 Sink / 模式 |
|----------|------------------|
| SQL      | `db.Exec`/`Query` 字符串拼接、`fmt.Sprintf` 拼 SQL、GORM 原生 SQL 拼接 |
| RCE/执行 | `exec.Command`、`os/exec`、`syscall` |
| SSRF     | `http.Get`/`http.Post`、`http.Client` 请求用户可控 URL、`net.Dial` |
| 文件     | `os.Open`/`os.Create`、`ioutil.ReadFile`/`WriteFile`、用户可控路径 |
| 反序列化 | `encoding/gob`、`json.Unmarshal` 到可导出结构、第三方反序列化库 |
| 逻辑/鉴权 | 中间件中的 `userID`/`role` 从 token 或 header 解析、Handler 内权限判断、条件分支 |

## 逻辑漏洞上下文

- 中间件与路由分组：明确哪些路由需要登录/权限，哪些为公开；对接收 `user_id`、`role`、`admin` 等参数的路由，在 Round 1 逻辑专项中追踪到业务层用法。
