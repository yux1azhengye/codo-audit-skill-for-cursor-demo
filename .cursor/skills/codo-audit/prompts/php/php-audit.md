# PHP 技术栈 Recon 与 Sink 策略

## 技术栈识别

- 依赖：`composer.json`、`composer.lock`。
- 框架：Laravel、Symfony、ThinkPHP、CodeIgniter、原生 PHP 等。
- ORM：Eloquent、Doctrine、PDO 封装。
- 模板：Blade、Twig、Smarty、原生 `include`/`require`。

## 路由与入口

- Laravel：`Route::get/post()`、`request()->input()`、`$request->get()`、路由中间件。
- Symfony：`@Route` 注解、`Request::query`/`request`、Security/Voter。
- 原生：`$_GET`、`$_POST`、`$_REQUEST`、`$_SERVER`、`$_COOKIE`、`$_FILES`。

## 常见 Sink（供 Round 1 选型）

| 类型     | 典型 Sink / 模式 |
|----------|------------------|
| SQL      | `mysql_query`、`mysqli_query` 拼接、PDO `query`/`exec` 拼接、`DB::raw()` 拼接、Eloquent 原生拼接 |
| RCE/执行 | `eval`、`assert`、`preg_replace`/e 修饰符、`system`/`exec`/`passthru`/`shell_exec`、`create_function`、反序列化 `unserialize` |
| SSRF     | `file_get_contents`(url)、`fopen`(url)、`curl_exec` 用户可控 URL、`Http::get`（Laravel） |
| 文件     | `include`/`require`（可 LFI）、`file_get_contents`、`fopen`/`fwrite`、`move_uploaded_file`、用户可控路径 |
| 反序列化 | `unserialize`、Phar 反序列化、常见 POP 链 |
| 逻辑/鉴权 | 中间件 `auth`、Gate/Policy、Session 校验、`$_SESSION` 中 role/user_id、手动 if 鉴权 |

## 逻辑漏洞上下文

- 中间件与路由：Laravel `middleware`、Symfony Security；标注哪些路由需登录/权限；接收 `user_id`、`id`、`role`、`admin` 等参数的路由在业务层用法需在逻辑专项中追踪。
- 文件包含：`include($user_input)` 常与路径穿越、日志注入组合，需单独看是否可控。
