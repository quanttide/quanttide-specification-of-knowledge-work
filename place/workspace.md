# 工作区

工作区（Workspace）是工作的边界：材料、产物、工作流、任务都归属于它；区内工件与过程互相可见、可流转，区间彼此隔绝。

工作区隶属于工作台。工作区是逻辑单元，只认内容、不认位置——工件在物理上从哪来、落在哪，由平台在装载时决定；同一个工作区，资源可以来自多处、任意组合。

## 领域模型

### 字段

- `id`（UUID，只读）：标识，全局唯一。
- `name`（Slug，必选）：名称，用于编排、URL 等；在其所属工作台内唯一。
- `title`（String，必选）：标题。
- `description`（String，推荐）：描述。默认为空。
- `created_at`（Datetime，只读）：创建时间。
- `updated_at`（Datetime，只读）：最近更新时间。

### 关联

一个工作区包含零至多个工作流（Workflow）、任务（Task）、产物（Artifact）、材料（Material）；关联通过子资源端点暴露与遍历，不作为内嵌字段随工作区响应返回。

### 约束

- 归属唯一：任何材料、产物、工作流、任务必须且只能归属于一个工作区，工作区是它们的作用域；
- 区内互见：同一工作区内的工件与过程彼此可见、可相互引用与流转；
- 区间隔绝：工作区之间默认互不可见；跨区引用须经显式的导出或导入操作。

## 领域事件

### 事件

- 工作区已创建（Workspace Created）

### 约束

- 工作区已创建先于其内一切事件——材料已收录、工作流已创建、任务已启动或已完成、产物已生成或已验收，均以工作区已创建为前提；
- 事件负载至少携带工作区 `id`，供下游投影、汇总与审计使用。

## API 规格

工作区建模为 REST 资源。集合端点挂载于工作台之下，以体现一对多层级；单个端点挂载于根路径，便于跨工作台引用。

### 工作区资源

- `POST /workbenches/{workbench_id}/workspaces`：创建工作区。请求体含 `name`（必选）、`title`（必选）；`name` 在工作台内冲突时返回 `409 Conflict`；成功返回 `201 Created` 与资源表示，并触发”工作区已创建“；
- `GET /workbenches/{workbench_id}/workspaces`：列出工作台下的工作区，支持分页；
- `GET /workspaces/{id}`：读取单个工作区；
- `PATCH /workspaces/{id}`：更新 `title`。

### 子资源端点

以下端点分别列出区内对象，均支持分页，不在工作区响应中内嵌返回：

- `GET /workspaces/{id}/materials`
- `GET /workspaces/{id}/workflows`
- `GET /workspaces/{id}/tasks`
- `GET /workspaces/{id}/artifacts`

### 约束

- 服务无状态，不维护会话状态；
- 创建工作区等非幂等操作，客户端应提供幂等键（Idempotency-Key 头），网络重试不产生重复工作区。
