---
name: django-ninja-project-standard
description: Django Ninja 后端项目标准。用于新建、改造或评审 Django/Django Ninja/Python API 后端项目，默认采用 Django + Django Ninja + Celery + Redis + PostgreSQL + Docker Compose + uv + pytest + ruff，并优先沿用 Flowner 的分层、环境、路由、服务目录和验证约定；适用于用户要求 Django Ninja 项目、Python 后端骨架、带 Celery/Docker 的 API 服务，或未指定后端技术栈但需要采用本地默认后端标准的场景。
metadata:
  short-description: Django Ninja 后端标准骨架
---

# Django Ninja 项目标准

本 skill 用于把后端项目落到统一工程标准。除非用户明确要求其他技术栈，否则默认采用 Django + Django Ninja + Celery + Redis + PostgreSQL + Docker Compose + uv，并以 Flowner 当前架构作为本地默认参考实现。

## 前置规则

- 同时遵守 `project-workflow`：先理解现有仓库，再精确修改，最后做匹配范围的验证。
- 新建项目可直接生成标准骨架；已有项目只补齐缺失能力，保留既有业务代码、可用约定和已经稳定的路由。
- 如果用户要求与本标准冲突，先指出冲突并确认取舍，不要静默替换技术栈、目录布局或运行方式。
- 不要把本 skill 当作通用 Python 模板；非 Django Ninja 后端场景不使用。
- 在 Flowner 仓库中使用时，优先遵守 `AGENTS.md`、`.ai/memory.md`、`README.md` 和 `docs/backend-architecture.md` 的最新约定。

## 默认技术栈

- Python 3.10+、Django、Django Ninja
- PostgreSQL 主数据库；SQLite 仅允许作为本地 fallback
- Redis 作为 cache、Celery broker 和 result backend
- Celery worker 处理异步任务；需要周期任务时增加 Celery beat
- Docker Compose 管理 `app`、`worker`、`db`、`redis`，全栈项目可增加 `frontend`、`beat`、`nginx`
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
    │   │   ├── schemas/
    │   │   ├── services/
    │   │   └── tests/
    │   └── example/
    │       ├── api.py
    │       ├── apps.py
    │       ├── models.py
    │       ├── schemas.py
    │       ├── services.py 或 services/
    │       └── tests/
    └── config/
        ├── __init__.py
        ├── asgi.py
        ├── celery_app.py
        ├── settings.py
        ├── urls.py
        └── wsgi.py
```

允许按业务新增 app；默认保留 `common` 和 `example`。已有项目可以暂不具备完整 `deploy/` 内容，但应保留目录作为 nginx、postgres、redis 等部署配置入口。复杂业务 app 优先使用 `services/` 目录收敛服务层，简单示例 app 可保留单文件 `services.py`。

## Flowner 参考架构

在 Flowner 或同类项目中，推荐采用以下业务 app 形态：

- `apps.common`：登录、健康检查、提示词模板、AI 调用日志、认证、请求日志中间件和跨 app 纯 helper。
- `apps.example`：标准示例 app，提供最小 schema/service/api/test 链路，便于验证项目骨架。
- `apps.flow`：Flow 导入、预估、查询和 ETL 服务，核心表为 `flows`。
- `apps.profile`：IP 画像、组织视图、证据、关联查询和分阶段异步任务。
- `apps.query`：自然语言查询、图查询或其他 API 编排，允许 `models.py` 为空壳占位。

不要在新改造中恢复已经淘汰的旧体系，例如 `apps.intel`、`/api/ingestion/*`、`intel_flows`、`intel_flow_raw_records`、`sync_cursors` 等。

## 生成或改造流程

1. 判断项目形态：新建项目生成骨架；已有项目先审查目录、依赖、配置、测试、Docker 和运行方式。
2. 补齐根目录文件：`.env.example`、`.gitignore`、`.dockerignore`、`.gitattributes`、`README.md`、`docker-compose.yml`、`logs/.gitkeep`。
3. 补齐 `backend/`：`manage.py`、`pyproject.toml`、`pytest.ini`、`Dockerfile`。
4. 补齐 `backend/config/`：Django settings、URL、ASGI/WSGI、Celery 入口。
5. 补齐 `backend/apps/common/`：公共 schema、health API、认证与跨 app 能力、服务包和通用 helper。
6. 补齐 `backend/apps/example/`：最小业务示例，覆盖 model/schema/service/api/test 链路。
7. 按业务需要增加 `flow`、`profile`、`query` 等 app；大型逻辑下沉到 `services/`，API 层只做请求入口和响应组装。
8. 配置 `/api/`、`/api/docs`、`/api/openapi.json`、`/api/ops/health` 并确保可访问。
9. 执行当前环境能完成的最小验证，无法验证时说明原因和用户可复现命令。

## 分层职责

### 根目录

- `.env.example`：提交到 Git 的环境变量样例，不含真实密钥。
- `.env`：Docker Compose 默认环境文件，通常不提交。
- `.env.local`：本地直接运行环境文件，通常不提交。
- `docker-compose.yml`：定义 `app`、`worker`、`db`、`redis`，按需定义 `frontend`、`beat`、`nginx`。
- `logs/`：运行日志目录，只提交 `.gitkeep`。
- `README.md`：说明本地运行、Docker 运行、测试、健康检查和 API 文档地址。
- `docs/backend-architecture.md`：记录长期架构、路由、app 边界和重要约定。

### `backend/config/`

- `settings.py`：只放框架配置；从环境变量读取配置；不写业务逻辑。
- `urls.py`：挂载 Django Ninja API、docs、OpenAPI JSON 和必要的 Django URL。
- `celery_app.py`：Celery app 初始化入口，供 worker、beat 和 Django 共用。
- `asgi.py` / `wsgi.py`：标准 Django 入口，不夹带业务逻辑。
- `etl_table_profiles.json` 或类似配置：仅在存在外部数据源 ETL 时使用，避免把源表 profile 写死到服务代码中。

### `backend/apps/common/`

`common` 是项目级公共 app，不承载具体业务域。

- `api.py`：聚合登录、ops/health、AI 管理等公共路由入口。
- `schemas/`：通用响应、分页、错误、健康检查和公共数据结构。
- `capabilities/`：认证、权限、异常处理、响应封装、日志等跨 app 能力。
- `capabilities/utils.py`：与业务域无强绑定的纯 helper，例如文本清洗、去重、分页整数限制、日期窗口、aware datetime、JSON 安全转换、日志预览截断。
- `services/`：公共服务能力；复杂能力可继续分包，例如 `services/ai/`。
- `tests/`：公共能力、登录、AI 管理和 health API 测试。

### `backend/apps/<business_app>/`

业务 app 围绕单一业务域组织。

- `api.py`：Django Ninja router 和请求入口；不要堆业务逻辑。
- `schemas.py` 或 `schemas/`：请求、响应和内部传输 schema。
- `services.py` 或 `services/`：用例编排、业务规则、事务边界、外部系统访问和复杂读取逻辑。
- `tasks.py`：Celery 异步任务入口，任务内调用 service，不直接堆业务流程。
- `models.py`：ORM 模型；只在需要持久化时创建，API/graph app 可保留空壳。
- `protocols/`：仅在需要显式抽象外部依赖或结构化接口时使用。
- `tests/`：覆盖 schema、service、API、任务编排和关键模型行为。

## API 与 schema 约定

- API 总入口固定为 `/api/`。
- 健康检查固定为 `/api/ops/health`，无需认证。
- 登录接口固定为 `/api/auth/login`，返回 DRF Token，无需认证。
- API 文档固定为 `/api/docs`。
- OpenAPI JSON 固定为 `/api/openapi.json`。
- 每个业务 app 暴露一个 `router`，在 `config/urls.py` 统一挂载。
- 请求/响应 schema 使用 Django Ninja schema；复杂业务输入不要直接暴露 ORM model。
- `example` app 至少提供一个可工作的示例接口，方便验证完整链路。
- 认证检查默认基于 Token；前端登录态可使用 `localStorage.auth_token`。

## Flowner 路由约定

- `/api/auth/login`：登录，无需认证。
- `/api/ops/health`：健康检查，无需认证。
- `/api/ai/`：提示词模板与 AI 调用日志。
- `/api/example/`：标准示例接口。
- `/api/flow/`：Flow 导入、预估、查询。
- `/api/profile/`：IP 画像、组织视图、证据、任务触发与重置。
- `/api/query/`：自然语言查询。

画像路由使用 `/api/profile/*`；Flow 路由使用 `/api/flow/*`。不要新增或恢复 `/api/ingestion/*`。

## Flow / ETL 约定

- 导入目标表是 `flows`，不是旧聚合流表。
- `Flow` 的唯一约束推荐使用 `(source_table, source_id)`，用于幂等去重。
- `POST /api/flow/import` 应先做 count preview；预估成功时响应消息附带“预计导入 X 条”。
- `GET /api/flow/import-count` 用于预估指定日期 / 来源表的可导入条数。
- 多来源表导入通过 Celery `group` 并发扇出。
- 源库读取使用 keyset 方式按 `id` 递增抓取，不使用大 offset。
- 快速 `COUNT` 失败时，回退到分批扫描计数。
- 来源表 profile 放在配置文件中维护，避免在 API 层写死。

## Profile / AI 约定

- 行为分析、智能分析、汇总生成都依赖统一结构化 LLM 调用，例如 `run_structured_llm`。
- 缺少 prompt、缺少 API key、模型调用异常或 Schema 校验失败时返回 `status=fallback`，不生成规则兜底画像数据。
- 智能分析与汇总生成直接写入 LLM 返回的用户画像字段，不使用本地 hint 逻辑补齐用户标签。
- 画像任务通过 `/api/profile/trigger/{stage}` 分阶段入队。
- 不保留或恢复单 IP 单日画像 management command，例如 `run_profile_day_analysis`。
- 破坏性接口必须显式标记风险；例如 `/api/profile/reset` 会清空画像相关数据和 AI 调用日志。

## 配置与环境变量

- 必备环境变量至少覆盖 `DJANGO_SECRET_KEY`、`DJANGO_DEBUG`、`DJANGO_ALLOWED_HOSTS`、数据库、Redis、Celery broker/result backend。
- 本地直接运行优先读取仓库根目录 `.env.local`；容器运行使用仓库根目录 `.env`；示例值维护在 `.env.example`。
- `POSTGRES_PASSWORD` 为空时可回退到 SQLite，仅作为本地开发 fallback。
- 外部源库读取使用 `SOURCE_DATABASE_URL` 或等价配置，不与主业务库混用。
- Celery broker / backend 默认使用 `REDIS_URL`。
- 容器内文件路径类变量必须使用绝对路径，例如 `MAXMIND_GEOIP2_CITY_DB_PATH`。
- 生产相关默认值必须保守：`DJANGO_DEBUG` 默认关闭，密钥缺失时应显式失败或只允许开发 fallback。
- 不提交 `.env`、`.env.local`、真实密钥、令牌、密码或私钥。

## Docker 与依赖

- `Dockerfile` 基于 `python:3.10-slim` 或更高兼容版本，安装 uv，并通过 `uv sync` 安装依赖。
- `app` 使用 `python manage.py runserver 0.0.0.0:8000` 作为开发默认命令。
- `worker` 复用后端镜像并运行 Celery worker。
- `beat` 如存在，运行 `celery -A config.celery_app beat`，需要 `django_celery_beat` 时使用数据库 scheduler。
- `db` 使用 PostgreSQL；`redis` 使用 Redis。
- `pyproject.toml` 至少包含 Django、Django Ninja、Celery、Redis client、PostgreSQL driver、pytest、pytest-django、ruff。

## 验证标准

优先执行当前环境下能完成的最小验证，常用命令按相关性选择：

1. `cd backend && uv run ruff check .`
2. `cd backend && uv run pytest`
3. `cd backend && uv run python manage.py check`
4. `docker compose config --quiet` 或 `docker-compose config --quiet`
5. `docker compose run --rm app pytest` 或 `docker-compose run --rm app pytest`
6. 访问或测试 `/api/ops/health`

在 Flowner 中，常用组合验证为：

```bash
cd backend
POSTGRES_PASSWORD= uv run ruff check .
POSTGRES_PASSWORD= uv run pytest apps/common/tests apps/flow/tests apps/query/tests apps/profile/tests apps/example/tests
POSTGRES_PASSWORD= uv run python manage.py check
```

如果依赖未安装、Docker 不可用或网络受限，说明未执行命令、失败原因、当前风险和用户可复现的下一步。

## 文档同步

更新仓库结构、路由、启动方式、环境变量或长期业务约定后，同步检查并按需修改：

- `README.md`
- `docs/backend-architecture.md`
- `AGENTS.md`
- `.ai/memory.md`
- `.env.example`

`.ai/memory.md` 只记录稳定且长期有价值的约定，不记录临时调试过程、测试日志或一次性草稿。

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
- FastAPI、Flask、DRF-only 或纯 Python 模板。
- 省略 Celery、Redis、PostgreSQL、Docker Compose，除非用户明确要求。
- 把 health 做成独立业务 app。
- 让 `common` 承载具体业务域。
- 在 `settings.py`、`api.py` 中堆积业务逻辑。
- 在 Flowner 中恢复 `intel` / `/api/ingestion/*` 旧体系。

## 维护要求

- 本 skill 只维护 Django Ninja 后端标准；通用编码流程放到 `project-workflow`。
- 更新规则时保持描述、目录结构、硬约束和验证步骤一致。
- 从 Flowner 提炼规则时，只沉淀稳定架构约定，不复制一次性业务数据、密钥或临时调试结论。
- 如果增加模板、脚本或参考资料，放入本 skill 的 `assets/`、`scripts/` 或 `references/`，并说明何时读取或执行。
