# 全局 Skills 说明

用于说明当前全局 skills 的职责边界、默认加载策略与维护原则。

## 当前保留的 skills

### 1. `project-workflow`
负责项目级任务流程：
- 新任务开始前读取稳定上下文
- 编码前检索现有实现与复用模式
- 任务中决定何时连续推进、何时暂停确认
- 改动后进行本地验证
- 按需回写长期有效的项目记忆

### 2. `coding-rule`
负责通用编码行为：
- 不假设，不隐藏困惑
- 简单优先
- 精确修改
- 用可验证目标驱动实现

### 3. `django-ninja-project-standard`
负责定义 Python + Django + Django Ninja 新项目开发标准。

## 依赖机制

通过 skill 元数据中的 `requires` 字段实现自动依赖：

| Skill | requires |
|-------|----------|
| `django-ninja-project-standard` | `project-workflow`, `coding-rule` |
| `project-workflow` | `coding-rule` |
| `coding-rule` | (base，无依赖) |

加载时会自动递归加载所有依赖。

## 默认加载策略

- 新建 Django Ninja 项目：加载 `django-ninja-project-standard`
- 仓库内开发任务：加载 `project-workflow`
- 小范围代码修改：加载 `coding-rule`

## 分工原则

- `project-workflow` 管流程
- `coding-rule` 管编码行为
- 专项标准 skill 只在对应技术场景启用