# Node.js 技术栈 Recon 与 Sink 策略

## 技术栈识别

- 依赖：`package.json`、`package-lock.json`、`yarn.lock`。
- 框架：Express、Koa、Fastify、NestJS、Hapi、Next.js（API routes）等。
- ORM/DB：Sequelize、TypeORM、Prisma、mongoose、`pg`/`mysql2` 原生。
- 模板：EJS、Pug、Handlebars、React SSR。

## 路由与入口

- Express：`app.get/post()`、`req.query`、`req.body`、`req.params`、中间件。
- Koa：`ctx.query`、`ctx.request.body`、`ctx.params`。
- NestJS：`@Get()`/`@Post()`、`@Query()`/`@Body()`/`@Param()`、Guard/Interceptor。

## 常见 Sink（供 Round 1 选型）

| 类型     | 典型 Sink / 模式 |
|----------|------------------|
| SQL      | 原生 `query(sql)` 拼接、Sequelize `query()` 拼接、knex 拼接、`${var}` 拼入 SQL |
| RCE/执行 | `child_process.exec`/`execSync`、`eval`、`new Function()`、模板引擎未转义（EJS 等）、反序列化（node-serialize、pickle 等） |
| SSRF     | `http.get`/`request`、`axios`、`fetch`、`got` 等使用用户可控 URL |
| 文件     | `fs.readFile`/`writeFile`、`path.join` 未校验、`sendFile`、用户可控路径、路径穿越 |
| 反序列化 | `JSON.parse` 后未校验即用、`node-serialize`、`safe-eval` 滥用 |
| 逻辑/鉴权 | Passport、JWT 中间件、`req.user`、角色/权限中间件、手动 `userId`/`role` 与资源归属校验 |

## 逻辑漏洞上下文

- 中间件顺序：认证/授权中间件是否覆盖所有敏感路由；接收 `userId`、`id`、`role`、`admin` 等参数的路由在 Service 层用法需在逻辑专项中追踪。
- 异步与回调：注意回调中权限校验是否在正确的异步链上，避免竞态与绕过。
