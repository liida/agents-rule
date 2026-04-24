---
name: django-ninja-project-standard
description: Django Ninja 项目标准。用于新建、改造或评审 Django/Django Ninja/Python API 后端项目，以及可选的配套 frontend-admin 管理前端；默认采用 Django + Django Ninja + Celery + Redis + PostgreSQL + Docker Compose + uv + pytest + ruff，前端可采用 Vue 3 + Vite + Vue Router + Axios；适用于用户要求 Django Ninja 项目、Python 后端骨架、带 Celery/Docker 的 API 服务，或未指定后端技术栈但需要采用本地默认项目标准的场景。
metadata:
  short-description: Django Ninja 项目标准骨架
---

# Django Ninja 项目标准

本 skill 用于把 Django Ninja 后端项目落到统一工程标准，并在需要时补齐配套 `frontend-admin/` 管理前端约定。除非用户明确要求其他技术栈，否则后端默认采用 Django + Django Ninja + Celery + Redis + PostgreSQL + Docker Compose + uv。

## 前置规则

- 同时遵守 `project-workflow`：先理解现有仓库，再精确修改，最后做匹配范围的验证。
- 新建项目可直接生成标准骨架；已有项目只补齐缺失能力，保留既有业务代码、可用约定和已经稳定的路由。
- 如果用户要求与本标准冲突，先指出冲突并确认取舍，不要静默替换技术栈、目录布局或运行方式。
- 不要把本 skill 当作通用 Python 模板；非 Django Ninja 后端场景不使用。
- 若仓库内存在 `AGENTS.md`、项目 README、架构文档或项目记忆，优先遵守这些更贴近当前项目的约定。

## 默认技术栈

- Python 3.10+、Django、Django Ninja
- PostgreSQL 主数据库；SQLite 仅允许作为本地开发 fallback
- Redis 作为 cache、Celery broker 和 result backend
- Celery worker 处理异步任务；需要周期任务时增加 Celery beat
- Docker Compose 管理 `app`、`worker`、`db`、`redis`，全栈项目可增加 `frontend`、`beat`、`nginx`
- uv 管理依赖，`pyproject.toml` 作为依赖和工具配置入口
- pytest 做测试，ruff 做静态检查
- 配套管理前端可选 Vue 3、Vite、Vue Router、Axios；已有前端项目优先沿用现有栈

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
├── frontend-admin/              # 可选：后台管理前端
│   ├── package.json
│   ├── index.html
│   ├── vite.config.js
│   ├── public/
│   └── src/
│       ├── App.vue
│       ├── main.js
│       ├── router/
│       │   └── index.js
│       ├── composables/
│       │   └── useApi.js
│       ├── theme/
│       └── views/
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

允许按业务新增 app；默认保留 `common` 和 `example`。已有项目可以暂不具备完整 `deploy/` 内容，但应保留目录作为 nginx、postgres、redis 等部署配置入口。复杂业务 app 优先使用 `services/` 目录收敛服务层，简单示例 app 可保留单文件 `services.py`。需要后台页面时可增加 `frontend-admin/`，但本 skill 的核心仍是 Django Ninja 后端标准。

## App 边界

- `apps.common`：项目级公共能力，例如健康检查、认证、权限、请求日志、中间件、共享 schema、通用 helper、外部能力适配器等；不承载具体业务域。
- `apps.example`：最小业务示例 app，提供 model/schema/service/api/test 链路，便于验证骨架和新项目约定。
- `apps.<business_app>`：具体业务域 app，按业务语言命名；API 层只做请求入口、鉴权、参数校验和响应组装，业务逻辑下沉到 `services/`。
- 跨 app 复用逻辑优先放入 `common` 的明确子包；不要让业务 app 互相深度 import 内部实现。
- 大型业务 app 可拆分 `services/`、`selectors/`、`tasks.py`、`repositories.py`、`integrations/`，但不要为了模板完整性创建空抽象层。

## 代码风格与注释

- 默认写中文注释：代码注释、docstring、`db_comment`、schema description 和面向用户的日志说明优先使用中文；代码标识符、API 字段名、数据库字段名保持英文或项目既有命名。
- 尽量覆盖每个方法：公开函数、service 函数、API handler、Celery 任务、复杂私有 helper、前端 composable 方法和重要组件方法都应有简短中文 docstring 或方法说明，说明“做什么、关键输入、返回/副作用”。
- 简单 getter、setter、生命周期钩子、事件转发或一眼可读的短小内部函数可不强制逐行注释，但所在文件应通过方法名或相邻说明保持可理解。
- 注释应解释业务意图、边界条件、幂等策略、异常处理和跨系统约定；不要把每一行赋值、分支或 ORM 调用机械翻译成中文。
- 数据库字段应优先使用 `db_comment` 记录稳定业务含义；对外 schema 字段使用 `Field(description=...)` 记录 API 文档含义。
- 避免遗留注释、TODO、临时调试说明和已失效的业务术语；重命名、删除接口或改变语义时同步更新注释、schema description、README 和测试断言。
- 日志不是注释替代品：长任务和关键调度路径要有明确开始、结束、跳过、失败状态日志；日志文本按项目语言保持一致。

## Model 使用要求

- Django model 只表达持久化数据、约束、索引和轻量展示方法；不要把跨表编排、外部 API 调用、AI 调用、导入流程或复杂统计塞进 model 方法。
- 所有业务表显式设置 `db_table`，使用稳定、可迁移、可读的表名；不要把一次性历史表名、内部实现名或过期业务名扩散到新 model。
- 主键默认使用 `BigAutoField` 或项目既有统一主键策略；时间字段按项目约定使用 timezone-aware `DateTimeField`，创建时间常用 `auto_now_add=True`。
- 重要字段补充 `db_comment`；字段约束在类型、`null`、`blank`、`default`、`choices` 中表达清楚，避免用 service 里的隐式约定代替数据库语义。
- JSON 字段默认使用 `default=dict` 或 `default=list`，不要使用可变字面量；JSON 只承载半结构化扩展数据，核心筛选、关联和排序字段应拆成普通列并建索引。
- 唯一性和查询性能通过 `UniqueConstraint`、`Index`、`db_index=True` 等显式声明；常见筛选、排序、外键或时间窗口字段要按真实查询路径设计索引。
- `app_label` 仅在确有必要时显式设置；常规 app model 不需要重复声明。`__str__` 保持轻量，不触发额外查询。
- model 变更必须考虑迁移、历史数据和回滚；已有项目中删除字段、改表名、改唯一约束前先查引用、迁移和测试。

## Schema 使用要求

- Django Ninja schema 负责 API 输入输出契约，不等同于 ORM model；外部接口不要默认暴露完整 ORM 字段，尤其是内部状态、原始载荷、密钥、调试字段和历史兼容字段。
- 请求 schema、列表项 schema、详情项 schema、响应 schema 分开命名，常用后缀为 `Request`、`Item`、`Response`；列表响应显式包含 `total`、`limit`、`offset`、`items` 等分页信息。
- 每个对外字段使用 `pydantic.Field(description=...)` 写清楚含义；可选字段显式设置 `default=None`，列表或 dict 使用 `default_factory`，不要共享可变默认值。
- API 时间、日期、ID、状态等字段格式必须稳定并在 description 中说明；如果前端或外部系统依赖字符串格式，service 层统一转换，不让 API 随 ORM 序列化细节漂移。
- Schema 字段名应表达对外业务语言，避免泄露内部临时命名；重命名时同步前端、文档、测试和 prompt/示例，必要时提供兼容期。
- 复杂嵌套响应应拆分为可复用子 schema；但不要为了模板完整性创建没有实际复用价值的空 schema 层。
- `apps.common.schemas` 放通用响应、错误、分页、认证、健康检查等跨 app 契约；业务 schema 留在对应业务 app，避免 `common` 反向依赖业务 app。
- Service 层返回普通 dict、dataclass 或明确结构均可，但 API 层必须最终组装为 schema 契约；不要让 ORM QuerySet 或 model 实例直接穿透到对外响应。

## 前端 Admin 约定

当项目需要配套管理前端时，默认采用 `frontend-admin/`，推荐 Vue 3 + Vite + Vue Router + Axios；已有项目使用其他前端栈时，优先沿用现有技术栈，不为套模板强行迁移。

- `src/router/index.js` 集中维护路由；每个页面路由设置稳定 `name`，鉴权页面使用 `meta.requiresAuth` 或项目既有等价机制。
- `src/App.vue` 只承载应用壳、导航、登录态入口和 `router-view`；页面业务逻辑放到 `views/`，共享状态或请求逻辑放到 `composables/` 或 store。
- `src/composables/useApi.js` 或等价 API client 统一封装 `baseURL`、认证 token、请求头、超时、401 处理和错误传播；页面不要重复拼装底层 axios 配置。
- `VITE_API_URL` 作为前端访问后端的默认环境变量；开发环境可指向 `http://localhost:8000`，容器或部署环境通过 `.env` / `.env.example` 明确配置。
- 页面目录按路由和业务域命名，例如 `FlowListView.vue`、`FlowDetailView.vue`、`ProfileOrgView.vue`；详情页和列表页分开，避免单个页面混入过多状态。
- 导航高亮优先使用 `$route.name` 或明确的 route-name 集合；只有天然层级路由才使用 `path.startsWith(...)`，避免同前缀的兄弟页面同时高亮。
- 前端字段名、筛选参数、分页参数必须跟后端 schema 契约一致；常用分页参数为 `limit`、`offset`，排序参数为 `sort_by`、`sort_order`，不要在页面中发明后端未支持的参数。
- 日期、时间、状态、枚举和错误提示在页面层保持用户可读；如果需要选择日期，使用真实可交互控件，不要只展示不可编辑文本。
- 长任务触发页面要显示触发成功、任务 ID、运行中、结束、跳过或失败等状态；后端任务日志和前端提示语言应保持一致。
- 图表、地图、大列表等重组件按需加载；大数据列表优先使用后端分页、筛选和排序，不在前端一次性拉全量再过滤。
- 删除、改名或新增后端接口时，同步检查前端调用、菜单入口、路由、空状态、错误提示和构建结果。

## 生成或改造流程

1. 判断项目形态：新建项目生成骨架；已有项目先审查目录、依赖、配置、测试、Docker 和运行方式。
2. 补齐根目录文件：`.env.example`、`.gitignore`、`.dockerignore`、`.gitattributes`、`README.md`、`docker-compose.yml`、`logs/.gitkeep`。
3. 补齐 `backend/`：`manage.py`、`pyproject.toml`、`pytest.ini`、`Dockerfile`。
4. 补齐 `backend/config/`：Django settings、URL、ASGI/WSGI、Celery 入口。
5. 补齐 `backend/apps/common/`：公共 schema、health API、认证与跨 app 能力、服务包和通用 helper。
6. 补齐 `backend/apps/example/`：最小业务示例，覆盖 model/schema/service/api/test 链路。
7. 按业务需要增加业务 app；大型逻辑下沉到 `services/`，API 层只做请求入口和响应组装。
8. 如需要前端 admin，补齐 `frontend-admin/` 的路由、API client、页面入口、环境变量和构建脚本。
9. 配置 `/api/`、`/api/docs`、`/api/openapi.json`、`/api/ops/health` 并确保可访问。
10. 执行当前环境能完成的最小验证，无法验证时说明原因和用户可复现命令。

## 分层职责

### 根目录

- `.env.example`：提交到 Git 的环境变量样例，不含真实密钥。
- `.env`：Docker Compose 默认环境文件，通常不提交。
- `.env.local`：本地直接运行环境文件，通常不提交。
- `docker-compose.yml`：定义 `app`、`worker`、`db`、`redis`，按需定义 `frontend`、`beat`、`nginx`。
- `logs/`：运行日志目录，只提交 `.gitkeep`。
- `README.md`：说明本地运行、Docker 运行、测试、健康检查和 API 文档地址。
- `docs/backend-architecture.md`：如项目存在长期架构文档，记录路由、app 边界、异步任务和重要约定。

### `frontend-admin/`（可选）

- `package.json`：定义 `dev`、`build`、`preview`，按项目需要定义 `lint`。
- `vite.config.js`：Vite 配置入口；不要把真实后端地址、密钥或部署私有信息写死在配置里。
- `src/router/`：路由表、路由名、鉴权元信息和必要的导航守卫。
- `src/composables/`：封装 API client、通用查询状态、认证状态等可复用逻辑。
- `src/views/`：页面级组件，负责页面状态、表单、列表、详情和图表组合。
- `src/theme/`：主题 token、全局样式和通用视觉变量；页面内样式只放局部差异。

### `backend/config/`

- `settings.py`：读取环境变量，配置 apps、数据库、Redis/cache、Django Ninja、日志、中间件、CORS、Celery。
- `urls.py`：创建或挂载 Ninja API，统一注册 `/api/` 下的路由，暴露 docs 和 openapi。
- `celery_app.py`：设置 `DJANGO_SETTINGS_MODULE`，创建 Celery app，读取 Django settings 中的 `CELERY_` 配置并 autodiscover tasks。
- `asgi.py` / `wsgi.py`：保持标准 Django 入口，避免写业务逻辑。

### `backend/apps/common/`

`common` 是项目级公共 app，不承载具体业务域。

推荐内容：
- `api.py`：健康检查、登录或公共运维端点。
- `schemas/`：通用响应、错误、分页、认证 schema。
- `services/`：认证、日志、外部 API 客户端、共享能力封装。
- `capabilities/`：可插拔能力，如 AI、地理位置、对象存储、消息通知等。
- `tests/`：覆盖 health、认证和公共能力的稳定契约。

### `backend/apps/<business_app>/`

- `models.py`：只描述该业务域的数据模型和约束。
- `schemas.py` 或 `schemas/`：Django Ninja 输入输出 schema；复杂业务输入不要直接暴露 ORM model。
- `api.py`：路由、鉴权、参数校验、调用服务层、响应组装。
- `services.py` 或 `services/`：业务逻辑、事务、外部系统编排。
- `tasks.py`：Celery 异步任务入口；长任务要有可观测日志和幂等策略。
- `tests/`：至少覆盖关键 API、服务层和任务调度逻辑。

## 路由约定

- `/api/auth/login`：如项目需要内置登录，登录端点无需认证。
- `/api/ops/health`：健康检查，无需认证。
- `/api/example/`：标准示例接口。
- `/api/<business-domain>/`：业务 API，路径使用稳定业务名，不使用内部实现名。
- `/api/docs`：交互式 API 文档。
- `/api/openapi.json`：OpenAPI schema。

业务路由应集中在 `backend/config/urls.py` 或明确的 API registry 中注册。删除、改名或新增路由时，同步 README、架构文档、前端调用和测试。

## 服务层与异步任务

- API 层不要堆积业务逻辑；复杂规则、事务和外部系统调用放入 service。
- Service 函数应使用清晰输入输出，避免依赖 request 对象。
- 长任务使用 Celery；需要并发 fan-out 时可用 Celery `group`，但要避免闭包或 generator 导致重复参数问题。
- 任务应具备幂等策略、明确日志、失败状态记录和可重试边界。
- 批量读取外部数据时优先 keyset/page-token 方式；大 offset 只在数据量可控时使用。
- 外部数据源、字段映射、导入 profile 等项目特定配置优先放在配置文件或 settings 中，不写死在 API 层。

## 配置与环境变量

- 必备环境变量至少覆盖 `DJANGO_SECRET_KEY`、`DJANGO_DEBUG`、`DJANGO_ALLOWED_HOSTS`、数据库、Redis、Celery broker/result backend。
- 本地直接运行优先读取仓库根目录 `.env.local`；容器运行使用仓库根目录 `.env`；示例值维护在 `.env.example`。
- `POSTGRES_PASSWORD` 为空时可回退到 SQLite，仅作为本地开发 fallback。
- 外部源库读取使用独立的 `*_DATABASE_URL` 或等价配置，不与主业务库混用。
- Celery broker / backend 默认使用 `REDIS_URL`。
- 容器内文件路径类变量必须使用绝对路径。
- 生产相关默认值必须保守：`DJANGO_DEBUG` 默认关闭，密钥缺失时应显式失败或只允许开发 fallback。
- 不提交 `.env`、`.env.local`、真实密钥、令牌、密码或私钥。

## Docker 与依赖

- `Dockerfile` 基于 `python:3.10-slim` 或更高兼容版本，安装 uv，并通过 `uv sync` 安装依赖。
- `app` 使用 `python manage.py runserver 0.0.0.0:8000` 作为开发默认命令。
- `worker` 复用后端镜像并运行 Celery worker。
- `beat` 如存在，运行 `celery -A config.celery_app beat`，需要 `django_celery_beat` 时使用数据库 scheduler。
- `db` 使用 PostgreSQL；`redis` 使用 Redis。
- `pyproject.toml` 至少包含 Django、Django Ninja、Celery、Redis client、PostgreSQL driver、pytest、pytest-django、ruff。
- 如存在 `frontend-admin/`，前端默认使用 Node + Vite；`package.json` 至少包含 Vue、Vue Router、Axios 和 Vite 相关依赖，按实际页面需要增加图表、日期控件、状态管理等依赖。

## 验证标准

优先执行当前环境下能完成的最小验证，常用命令按相关性选择：

1. `cd backend && uv run ruff check .`
2. `cd backend && uv run pytest`
3. `cd backend && uv run python manage.py check`
4. `docker compose config --quiet` 或 `docker-compose config --quiet`
5. `docker compose run --rm app pytest` 或 `docker-compose run --rm app pytest`
6. 访问或测试 `/api/ops/health`
7. 如修改 `frontend-admin/`：`cd frontend-admin && npm run build`
8. 如前端配置了 lint：`cd frontend-admin && npm run lint`

如果项目依赖 PostgreSQL 但本地只需静态检查，可按项目约定临时设置空密码或 SQLite fallback；不要把某个业务项目的测试路径写成本 skill 的默认验证命令。

如果依赖未安装、Docker 不可用或网络受限，说明未执行命令、失败原因、当前风险和用户可复现的下一步。

## 文档同步

更新仓库结构、路由、启动方式、环境变量或长期业务约定后，同步检查并按需修改：

- `README.md`
- `docs/backend-architecture.md`
- `AGENTS.md`
- `.ai/memory.md`
- `.env.example`
- `frontend-admin/src/router/index.js`
- `frontend-admin/src/composables/useApi.js`

`.ai/memory.md` 只记录稳定且长期有价值的约定，不记录临时调试过程、测试日志或一次性草稿。若目标项目没有这些文档，不要为满足模板强行创建全部文件；只更新实际存在或本次新增后有维护价值的文档。

## 硬约束

必须保留：
- 目录：`backend/`、`backend/config/`、`backend/apps/`
- App：`common`、`example`
- 入口：`backend/manage.py`、`backend/config/celery_app.py`、`backend/pyproject.toml`
- 服务：`app`、`worker`、`db`、`redis`
- 路径：`/api/`、`/api/ops/health`、`/api/docs`、`/api/openapi.json`
- 日志目录：`logs/`

如项目包含 `frontend-admin/`，必须保留：
- 前端入口：`frontend-admin/package.json`、`frontend-admin/src/main.js`、`frontend-admin/src/App.vue`
- 前端路由：`frontend-admin/src/router/`
- API client：`frontend-admin/src/composables/useApi.js` 或项目既有等价封装

禁止默认改成：
- `src/` 布局或把 Django 项目直接放到仓库根目录。
- FastAPI、Flask、DRF-only 或纯 Python 模板。
- 省略 Celery、Redis、PostgreSQL、Docker Compose，除非用户明确要求。
- 把 health 做成独立业务 app。
- 让 `common` 承载具体业务域。
- 在 `settings.py`、`api.py` 中堆积业务逻辑。
- 默认生成或恢复任何特定项目的历史 app、历史路由、业务表名或旧接口。
- 在前端页面中绕过统一 API client、硬编码生产地址或复制多份认证/401 处理逻辑。
- 用宽泛 path 前缀匹配导致兄弟菜单同时高亮，除非该路由确实是父子层级。

## 项目特定规则

- 本 skill 只定义通用 Django Ninja 后端标准和可选管理前端工程约定；业务 app 名称、前端页面、路由、表名、外部数据源、AI 流程、导入策略等必须来自目标项目上下文或用户需求。
- 如果在某个已有项目中已经沉淀了额外约定，把它们当作该项目的本地约定处理，不要复制到其他项目。
- 只有当用户明确要求“参考某项目实现”或当前仓库就是该项目时，才使用该项目的历史经验作为辅助判断。
- 从具体项目提炼规则时，只沉淀可跨项目复用的工程约定，不复制一次性业务数据、密钥、临时调试结论或已废弃接口。

## 维护要求

- 本 skill 以后端标准为核心，只维护 Django Ninja 后端和可选 `frontend-admin/` 管理端的工程约定；通用编码流程放到 `project-workflow`。
- 更新规则时保持描述、目录结构、硬约束和验证步骤一致。
- 避免在 frontmatter、标题和默认流程中写入某个业务项目名；项目名只应出现在明确标注的“项目特定规则”或外部参考资料中。
- 如果增加模板、脚本或参考资料，放入本 skill 的 `assets/`、`scripts/` 或 `references/`，并说明何时读取或执行。
