# Python 技术栈 Recon 与 Sink 策略

## 技术栈识别

- 依赖：`requirements.txt`、`setup.py`、`pyproject.toml`。
- 框架：Django、Flask、FastAPI、Tornado。
- ORM：Django ORM、SQLAlchemy、Peewee。
- 模板：Jinja2、Django Template。

## 路由与入口

- Flask：`@app.route()`、`request.args`、`request.form`、`request.json`。
- Django：`urls.py` 与 view 映射、`request.GET`/`POST`、`request.body`。
- FastAPI：`@app.get/post`、`Query()`、`Body()`、依赖注入。

## 常见 Sink（供 Round 1 选型）

| 类型     | 典型 Sink / 模式 |
|----------|------------------|
| SQL      | 原始 SQL 拼接、`cursor.execute(sql % ...)`、ORM `raw()`/`extra()` 拼接、SQLAlchemy `text()` 拼接 |
| RCE/执行 | `os.system`、`subprocess`、`eval`、`exec`、`pickle.loads`、模板注入（Jinja2） |
| SSRF     | `requests.get/post`、`urllib.request`、`httpx` 等使用用户可控 URL |
| 文件     | `open()`、`pathlib.Path` 读写、用户可控路径、`send_file` 路径 |
| 反序列化 | `pickle.loads`、`yaml.load`（未 SafeLoader）、`marshal.loads` |
| 逻辑/鉴权 | `@login_required`、`@permission_required`、view 内 `request.user`、`user_id`/`is_staff` 等判断 |

## 逻辑漏洞上下文

- 装饰器与中间件：Django 中间件、Flask `before_request`、FastAPI 依赖；Recon 时标注哪些路由需要登录/权限，以及接收敏感参数的路由，供逻辑专项按「路由 → view → 业务逻辑」追踪。
