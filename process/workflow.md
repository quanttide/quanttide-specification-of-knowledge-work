# 工作流

工作流（Workflow）是过程的定义：一串有序的步骤，每步写着谁做与怎么算完。一份定义可以起多件工单，工单照着它走一遍（见 [工单](./work-order.md)）。

## 领域属性

### 字段

工作流落成一份 YAML 文件，文件名即工作流名：

- `id`（UUID，必选）：全球凭证，定义侧创建时生成，全局唯一，永不重发；工单封面的 `workflow_id` 查填认它。
- `name`（String，必选）：工作流名，在其所属工作区内唯一。
- `description`（String，推荐）：一句话说清这条工作流干什么，默认为空。


### 关联

工作流隶属工作区，被工单按名引用（见 [工作区](../place/workspace.md)）。

判据里的路径指向区内工件，相对路径的基准由平台给出。

### 约束

- 字段取值之外一律拒绝：出现不认识的字段、缺必填字段、取值不在枚举内，均视为不合语法；
- `rule` 判据必须且只能写一种判法；`agent` 或 `human` 判据只写 `description`，不带判法字段——前者是给智能体的判准，后者是留给人拍板的事项；
- 定义要能对着工作区核一遍：`path` 与 `file` 判据里的路径须在区内，`description` 里提到的小节须有 `contains` 判据覆盖；核对不访问文件系统，在不在由端侧判断。

## 领域事件

### 事件

- 工作流已创建（`WorkflowCreated`）

### 约束

- 事件负载至少携带工作流名与工作区 `id`，供下游投影、汇总与审计使用。

## API端点

工作流建模为 REST 资源，端点挂载于工作区之下：定义认名字，跨区互不可见。

### 工作流资源端点

- `POST /workbenches/{workbench}/workspaces/{workspace}/workflows`：创建工作流。
- `GET /workbenches/{workbench}/workspaces/{workspace}/workflows`：列出工作区下的工作流。
- `GET /workbenches/{workbench}/workspaces/{workspace}/workflows/{workflow}`：读取单个工作流。

### 子资源端点

工作流不设子资源端点：步骤与判据内嵌在定义里。

### 约束

- 创建工作流撞上区内同名即拒绝，不覆盖；
- 无状态、幂等键等 API 口径见 [工作区](../place/workspace.md)。
