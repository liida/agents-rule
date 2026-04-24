---
name: django-ninja-project-standard
description: Django Ninja 后端项目标准。用于新建、改造或评审 Django/Django Ninja/Python API 后端项目，默认采用 Django + Django Ninja + Celery + Redis + PostgreSQL + Docker Compose + uv + pytest + ruff，约束标准目录、配置、应用分层、API 路由、环境变量、Docker 服务和验证方式；适用于用户要求 Django Ninja 项目、Python 后端骨架、带 Celery/Docker 的 API 服务，或未指定后端技术栈但需要采用本地默认后端标准的场景。
metadata:
  short-description: Django Ninja 后端标准骨架
---

# Django Ninja 项目标准

本 skill 用于把后端项目落到统一工程标准。除非用户明确要求其他技术栈，否则默认采用 Django + Django Ninja + Celery + Redis + PostgreSQL + Docker Compose + uv。

## 前置规则

- 同时遵守 `project-workflow`：先理解现有仓库，再精确修改，最后做匹配范围的验证。
- 新建项目可直接生成标准骨架；已有项目只补齐缺失能力，保留既有业务代码和可用约定。
- 如果用户要求与本标准冲突，先指出冲突并确认取舍，不要静默替换技术栈或目录布局。
- 不要把本 skill 当作通用 Python 模板；非 Django Ninja 后端场景不使用。

## 默认技术栈

- Python 3.10+、Django、Django Ninja
- PostgreSQL 主数据库；SQLite 仅允许作为本地 fallback
- Redis 作为 cache、Celery broker 和 result backend
- Celery worker 处理异步任务
- Docker Compose 管理 `app`、`worker`、`db`、`redis`
- uv 管理依赖，`pyproject.toml` 作为依赖和工具配置入口
- pytest 做测试，ruff 做静态检查

## 标准目录

```text
.
├── .dockerignore
├── .env.example
├── .gitattributes
├── .gitignore
├── README.md
├── docker-compose.yml
├── logs/
│   └── .gitkeep
├── deploy/
│   └── README.md
└── backend/
    ├── Dockerfile
    ├── manage.py
    ├── pyproject.toml
    ├── pytest.ini
    ├── apps/
    │   ├── common/
    │   │   ├── api.py
    │   │   ├── apps.py
    │   │   ├── capabilities/
    │   │   ├── libs/
    │   │   ├── schemas/
    │   │   └── tests/
    │   └── example/
    │       ├── api.py
    │       ├── apps.py
    │       ├── models.py
    │       ├── schemas.py
    │       ├── services.py
    │       └── tests/
    └── config/
        ├── __init__.py
        ├── asgi.py
        ├── celery_app.py
        ├── settings.py
        ├── urls.py
        └── wsgi.py
```

允许按业务新增 app；不要删除 `common` 和 `example`。已有项目可以暂不具备完整 `deploy/` 内容，但应保留目录作为 nginx、postgres、redis 等部署配置入口。

## 生成或改造流程

1. 判断项目形态：新建项目生成骨架；已有项目先审查目录、依赖、配置、测试和运行方式。
2. 补齐根目录文件：`.env.example`、`.gitignore`、`.dockerignore`、`.gitattributes`、`README.md`、`docker-compose.yml`、`logs/.gitkeep`。
3. 补齐 `backend/`：`manage.py`、`pyproject.toml`、`pytest.ini`、`Dockerfile`。
4. 补齐 `backend/config/`：Django settings、URL、ASGI/WSGI、Celery 入口。
5. 补齐 `backend/apps/common/`：公共 schema、health API、跨 app 能力和底层支撑库。
6. 补齐 `backend/apps/example/`：最小业务示例，覆盖 model/schema/service/api/test 链路。
7. 配置 `/api/`、`/api/docs`、`/api/openapi.json`、`/api/ops/health` 并确保可访问。
8. 执行当前环境能完成的最小验证，无法验证时说明原因和用户可复现命令。

## 分层职责

### 根目录

- `.env.example`：提交到 Git 的环境变量样例，不含真实密钥。
- `.env`：Docker Compose 默认环境文件，通常不提交。
- `.env.local`：本地直接运行环境文件，通常不提交。
- `docker-compose.yml`：定义 `app`、`worker`、`db`、`redis`。
- `logs/`：运行日志目录，只提交 `.gitkeep`。
- `README.md`：说明本地运行、Docker 运行、测试、健康检查和 API 文档地址。

### `backend/config/`

- `settings.py`：只放框架配置；从环境变量读取配置；不写业务逻辑。
- `urls.py`：挂载 Django Ninja API、docs、OpenAPI JSON 和必要的 Django URL。
- `celery_app.py`：Celery app 初始化入口，供 worker 和 Django 共用。
- `asgi.py` / `wsgi.py`：标准 Django 入口，不夹带业务逻辑。

### `backend/apps/common/`

`common` 是项目级公共 app，不承载具体业务域。

- `api.py`：聚合 ops/health 等公共路由。
- `schemas/`：通用响应、分页、错误和健康检查 schema。
- `capabilities/`：认证、权限、异常处理、响应封装、日志等跨 app 能力。
- `libs/`：cache、tasks、clients、storage、utils 等底层支撑库。
- `tests/`：公共能力和 health API 测试。

### `backend/apps/<business_app>/`

业务 app 围绕单一业务域组织。

- `api.py`：Django Ninja router 和请求入口；不要堆业务逻辑。
- `schemas.py`：请求、响应和内部传输 schema。
- `services.py`：用例编排、业务规则和事务边界。
- `models.py`：ORM 模型；只在需要持久化时创建。
- `tests/`：覆盖 schema、service、API 和关键模型行为。

## API 与 schema 约定

- API 总入口固定为 `/api/`。
- 健康检查固定为 `/api/ops/health`。
- API 文档固定为 `/api/docs`。
- OpenAPI JSON 固定为 `/api/openapi.json`。
- 每个业务 app 暴露一个 `router`，在 `config/urls.py` 统一挂载。
- 请求/响应 schema 使用 Django Ninja schema；复杂业务输入不要直接暴露 ORM model。
- `example` app 至少提供一个可工作的示例接口，方便验证完整链路。

## 配置与环境变量

- 必备环境变量至少覆盖 `DJANGO_SECRET_KEY`、`DJANGO_DEBUG`、`DJANGO_ALLOWED_HOSTS`、数据库、Redis、Celery broker/result backend。
- 本地直接运行优先读取 `.env.local`；容器运行使用 `.env`；示例值维护在 `.env.example`。
- 生产相关默认值必须保守：`DJANGO_DEBUG` 默认关闭，密钥缺失时应显式失败或只允许开发 fallback。
- 不提交 `.env`、`.env.local`、真实密钥、令牌、密码或私钥。

## Docker 与依赖

- `Dockerfile` 基于 `python:3.10-slim` 或更高兼容版本，安装 uv，并通过 `uv sync` 安装依赖。
- `app` 使用 `python manage.py runserver 0.0.0.0:8000` 作为开发默认命令。
- `worker` 复用后端镜像并运行 Celery worker。
- `db` 使用 PostgreSQL；`redis` 使用 Redis。
- `pyproject.toml` 至少包含 Django、Django Ninja、Celery、Redis client、PostgreSQL driver、pytest、pytest-django、ruff。

## 验证标准

优先执行当前环境下能完成的最小验证，常用命令按相关性选择：

1. `ruff check .`
2. `pytest`
3. `python manage.py check`
4. `docker-compose config` 或 `docker compose config`
5. `docker-compose run --rm app pytest` 或 `docker compose run --rm app pytest`
6. 访问或测试 `/api/ops/health`

如果依赖未安装、Docker 不可用或网络受限，说明未执行命令、失败原因、当前风险和用户可复现的下一步。

## 硬约束

必须保留：
- 目录：`backend/`、`backend/config/`、`backend/apps/`
- App：`common`、`example`
- 入口：`backend/manage.py`、`backend/config/celery_app.py`、`backend/pyproject.toml`
- 服务：`app`、`worker`、`db`、`redis`
- 路径：`/api/`、`/api/ops/health`、`/api/docs`、`/api/openapi.json`
- 日志目录：`logs/`

禁止默认改成：
- `src/` 布局或把 Django 项目直接放到仓库根目录。
- FastAPI、Flask、DRF 或纯 Python 模板。
- 省略 Celery、Redis、PostgreSQL、Docker Compose，除非用户明确要求。
- 把 health 做成独立业务 app。
- 让 `common` 承载具体业务域。
- 在 `settings.py`、`api.py` 中堆积业务逻辑。

## 维护要求

- 本 skill 只维护 Django Ninja 后端标准；通用编码流程放到 `project-workflow`。
- 更新规则时保持描述、目录结构、硬约束和验证步骤一致。
- 如果增加模板、脚本或参考资料，放入本 skill 的 `assets/`、`scripts/` 或 `references/`，并说明何时读取或执行。
