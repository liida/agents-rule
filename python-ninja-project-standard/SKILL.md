---
name: python-ninja-project-standard
description: 定义参考 Flowner 风格的 Python + Django + Django Ninja 新项目起步标准，包含目录组织、文件职责、边界约定与默认开发实践。
compatibility: opencode
role: standard
---

# Python + Django Ninja 新项目开发标准

## 目标

为 Python + Django Ninja 新项目开始开发时提供统一的目录组织、文件职责、边界约定和默认实践。

默认不把 Python、Django、Django Ninja 版本写死到某一个固定小版本；应优先选择当前稳定、兼容、主流的版本组合，必要时再根据用户要求或项目约束固定版本。

默认风格参考以下约定：

- 后端目录使用 `backend/`
- 业务模块使用 `backend/apps/`
- Django 项目配置使用 `backend/config/`
- 依赖管理优先使用 `pyproject.toml + uv`
- 默认预留 `pytest`、`ruff`、Celery、Redis、Docker Compose
- 环境变量优先使用根目录 `.env.local` 与 `.env.docker`

## 何时使用

- 用户要求“新建一个 Python + Ninja 项目”
- 用户要求“给我一个 Django Ninja 项目标准”
- 用户要求“参考现有 backend/apps/config 风格初始化新服务”
- 用户要明确新项目开始开发时的默认目录组织和约定

## 常见可调整项

开始新项目时，通常需要先明确这些可调整项：

- `project_name`
- `python_package`
- `backend_dir`（默认 `backend`）
- `api_prefix`（默认 `/api`）
- `use_uv`（默认 `true`）
- `use_celery`（默认 `false`）
- `use_docker`（默认 `false`）
- `use_postgres`（默认 `true`）
- `allow_sqlite_fallback`（默认 `true`）
- `apps`（默认 `common`、`health`、`example`）

如用户未明确提供，可先按以下默认理解：

```yaml
project_name: my_service
python_package: my_service
backend_dir: backend
api_prefix: /api
use_uv: true
use_celery: false
use_docker: false
use_postgres: true
allow_sqlite_fallback: true
apps:
  - common
  - health
  - example
```

## 参考目录结构（含按需增强项）

以下目录结构包含常见增强项。实际应根据 `use_docker`、`use_celery`、`use_postgres` 等选项裁剪，而不是默认全部纳入。

```text
.
├── .dockerignore
├── .gitattributes
├── .env.docker
├── .env.local
├── .gitignore
├── README.md
├── docker-compose.yml
└── backend/
    ├── Dockerfile
    ├── manage.py
    ├── pyproject.toml
    ├── pytest.ini
    ├── apps/
    │   ├── common/
    │   │   ├── apps.py
    │   │   ├── api.py
    │   │   ├── schemas.py
    │   │   ├── services.py
    │   │   └── tests/
    │   ├── health/
    │   └── example/
    │       ├── apps.py
    │       ├── api.py
    │       ├── models.py
    │       ├── schemas.py
    │       ├── services.py
    │       └── tests/
    └── config/
        ├── settings.py
        ├── urls.py
        ├── asgi.py
        ├── wsgi.py
        └── celery_app.py
```

## 目录与文件职责

### 根目录

- `.dockerignore`：Docker 构建时忽略无需进入镜像上下文的文件；仅在启用 Docker 时需要。
- `.gitattributes`：统一 Git 文本换行、二进制文件处理及特定文件属性。
- `.env.local`：本地开发环境变量文件。
- `.env.docker`：Docker 或容器运行时环境变量文件；仅在启用 Docker 时需要。
- `.gitignore`：忽略虚拟环境、缓存、环境变量文件、构建产物等无需提交的内容。
- `README.md`：项目说明、启动方式、开发命令、测试命令与常见注意事项。
- `docker-compose.yml`：本地容器编排文件；仅在启用 Docker 时需要。

### `backend/`

- `manage.py`：Django 管理入口，用于 `runserver`、`migrate`、`createsuperuser` 等命令。
- `pyproject.toml`：项目依赖与工具配置入口，优先用于管理 Python 依赖、pytest、ruff 等配置。
- `pytest.ini`：pytest 运行配置。
- `Dockerfile`：后端镜像构建文件；仅在启用 Docker 时需要。

### `backend/config/`

- `settings.py`：Django 配置入口，负责读取环境变量、注册 apps、中间件、数据库、日志等基础配置；不承载业务逻辑。
- `urls.py`：全局路由聚合入口，负责挂载 Django Ninja API、管理后台路由或其他顶层 URL。
- `asgi.py`：ASGI 部署入口。
- `wsgi.py`：WSGI 部署入口。
- `celery_app.py`：Celery 应用入口；仅在启用 Celery 时需要。

### `backend/apps/`

- `apps/` 用于存放业务 app，每个 app 应围绕单一业务域组织，而不是按技术类型堆叠。

### `backend/apps/<app>/`

- `apps.py`：Django app 注册配置。
- `api.py`：接口层，定义 Django Ninja Router、路由与请求入口。
- `schemas.py`：接口输入输出的数据结构定义，如请求体、响应体、序列化对象。
- `services.py`：业务逻辑层，负责用例编排、领域逻辑和可复用服务函数。
- `models.py`：数据模型定义；仅在该 app 需要数据库持久化时创建。
- `tests/`：该 app 的测试目录，优先放置与当前 app 对应的接口、服务、模型测试。

## 文件边界规则

- `api.py` 只负责接口定义、参数接收、调用服务层和返回响应，不应承载复杂业务逻辑。
- `schemas.py` 只定义数据结构与校验规则，不负责数据库访问和业务编排。
- `services.py` 负责业务逻辑，不负责注册路由，不应承担配置读取入口职责。
- `models.py` 只放 ORM 模型、模型字段、必要的模型级方法，不应承载接口层逻辑。
- `settings.py` 只放配置与环境变量读取，不放业务逻辑。
- `urls.py` 负责顶层路由聚合，不应堆积具体业务实现。
- `README.md` 应说明如何启动、测试、迁移与开发，而不是复制大量实现细节。

## 可选文件纳入条件

- `models.py`：只有在该业务 app 需要数据库模型时才创建。
- `services.py`：如业务逻辑足够简单，可保持极简；但只要存在可复用业务逻辑，就应放入该文件。
- `celery_app.py`：仅在 `use_celery=true` 时纳入项目结构。
- `.dockerignore`：仅在 `use_docker=true` 时纳入项目结构。
- `Dockerfile`、`docker-compose.yml`、`.env.docker`：仅在 `use_docker=true` 时纳入项目结构。
- PostgreSQL 相关配置：在 `use_postgres=true` 时作为默认数据库配置；若允许 SQLite fallback，应明确本地回退方式。
- `example` app：可作为演示或参考 app；如果用户明确要求极简结构，可不纳入项目结构。

## 项目分层

### 最小必需结构

在尽量精简、但仍可启动和继续开发的前提下，至少应包含：

- `.gitignore`
- `.gitattributes`
- `.env.local`
- `README.md`
- `backend/manage.py`
- `backend/pyproject.toml`
- `backend/pytest.ini`
- `backend/config/settings.py`
- `backend/config/urls.py`
- `backend/config/asgi.py`
- `backend/config/wsgi.py`
- 至少一个可工作的 app，例如 `health` 或 `common`
- 至少一个可访问的健康检查接口

### 推荐默认结构

在默认场景下，除最小必需结构外，推荐再包含：

- `common` app
- `health` app
- `example` app（作为演示或参考）
- `schemas.py`
- `services.py`
- 测试目录与基础测试用例
- PostgreSQL 配置与 SQLite fallback 说明

### 按需增强结构

只有在明确需要时才加入：

- `models.py`（需要持久化数据时）
- `celery_app.py`（需要异步任务时）
- `Dockerfile`
- `docker-compose.yml`
- `.dockerignore`
- `.env.docker`

应优先从“最小必需结构”出发，再根据用户需求补充“推荐默认结构”和“按需增强结构”，避免无条件堆叠所有文件。

## 默认开发约定

### 配置

- 配置全部从环境变量读取
- 数据库优先 PostgreSQL，可选回退 SQLite
- 不把业务逻辑写进 `settings.py`

### 版本策略

- 默认不要把 Python、Django、Django Ninja、pytest、ruff 等依赖钉死到过旧或过窄的单一版本。
- 如无用户明确要求，优先选择当前稳定、主流、彼此兼容的版本范围。
- 只有在以下情况才应显式固定版本：
  - 用户明确指定版本
  - 项目已有既定版本约束
  - 某个依赖存在明确兼容性边界，需要避免歧义
- 如果需要展示版本，优先用“推荐版本范围”或“最低兼容版本”表达，而不是默认写死到某个历史版本。

### 路由

- 使用 Django Ninja
- 默认统一挂载在 `/api/`
- 健康检查接口默认使用 `/api/ops/health`

### app 组织

每个业务 app 推荐至少包含：

- `apps.py`
- `api.py`
- `schemas.py`
- `services.py`（如有业务逻辑）
- `models.py`（如有数据模型）
- `tests/`

### 运行体验

- 优先提供 `uv sync` / `uv run` 命令
- 即便用户没配 PostgreSQL，也应允许先用 SQLite 启动
- Docker 是增强能力，不应成为本地最小启动前提
- Celery、Docker 等增强基础设施应按需启用，而不是默认成为最小结构的一部分

## 自包含要求

- 本 skill 应尽量自包含，不依赖额外模板清单、示例输入文件或目录树说明文件。
- 如需说明项目起步标准，应直接给出目录结构、关键文件职责、默认约定与验证方式。
- 不要求依赖同目录下的额外模板文件才能完成任务。

## 项目起步完成标准

一个符合该标准的新项目起步阶段，至少应满足：

在 `use_uv=true` 的默认场景下，至少应满足：

1. `uv sync`
2. `uv run python manage.py check`
3. `uv run python manage.py migrate`
4. `uv run pytest`
5. `uv run ruff check .`
6. `/api/ops/health` 可访问

若项目未使用 `uv`，应替换为等价的依赖安装、Django 检查、迁移、测试与静态检查命令。

## 禁止事项

- 不要只输出目录树
- 不要默认引入过重基础设施
- 不要强制 Docker 才能开发
- 不要把所有接口堆在一个文件里

## 维护要求

- 除 `SKILL.md` 外，不应继续维护额外说明文件、示例输入文件或模板索引文件。
- 若规则发生变化，应直接更新本文件，使其保持为单文件、自解释的 skill。
