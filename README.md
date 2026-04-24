# 本地 Skills 说明

此目录保存本机可用的自定义 Codex skills，用于沉淀稳定的项目工作流和专项工程标准。

## 当前 skills

### `project-workflow`

仓库级开发任务工作流：
- 任务开始前读取 `.ai/memory.md`、`AGENTS.md` 和相关上下文。
- 编码前理解现有实现、复用模式、命名风格和验证方式。
- 实现时保持简单优先、精确修改和风险可控。
- 完成后执行匹配范围的本地验证，并说明无法验证的部分。
- 有长期价值时更新项目记忆，避免记录敏感信息或临时日志。

### `django-ninja-project-standard`

Django Ninja 后端项目标准：
- 默认技术栈：Django + Django Ninja + Celery + Redis + PostgreSQL + Docker Compose + uv。
- 约束 `backend/`、`config/`、`apps/common/`、`apps/example/` 等目录边界。
- 固定 API 入口、健康检查、文档地址、环境变量和 Docker 服务名。
- 适用于新建、改造或评审 Django Ninja/Python API 后端项目。

## 使用关系

- 仓库内开发任务优先使用 `project-workflow`。
- Django Ninja 后端任务使用 `django-ninja-project-standard`，并同时遵守 `project-workflow`。
- 非 Django Ninja 场景不默认套用 Django 标准。

## 维护原则

- `SKILL.md` 放触发条件、关键流程和硬约束，保持简洁可执行。
- `agents/openai.yaml` 放 UI 元数据，需与 `SKILL.md` 的职责一致。
- 模板、脚本、参考资料分别放入 skill 目录下的 `assets/`、`scripts/`、`references/`。
- 不在 skills 中保存密钥、令牌、密码、私钥或项目敏感配置。
