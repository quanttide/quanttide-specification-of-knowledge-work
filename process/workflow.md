# 工作流

工作流（Workflow）是过程的定义：一串有序的步骤，每步写着谁做与怎么算完。一份定义可以起多件工作任务，工作任务照着它走一遍（见 [工作任务](./work-task.md)）。

## 领域属性

### 字段

工作流落成一份定义：

- `id`（UUID，派生）：工作流的全球凭证，按名造号、入账即锚，定义不写此字段（见 [凭证](#凭证)）。
- `name`（String，必选）：工作流名，在其所属工作区内唯一。
- `description`（String，推荐）：一句话说清这条工作流干什么，默认为空。
- `steps`（必选）：行程上的各站，按序排列；步骤字段见 [工作步骤](./work-step.md)。

### 凭证

工作流凭证即 `id`：工作任务封面、事件负载与 API 响应中的 `workflow_id` 即此凭证。按「工作区 id + 工作流名」造号。

- 同一工作区内名字唯一；跨工作区同名各得各号，账目不串；
- 首次入账即锚定，终身不变——锚定前凭证只是名字的推论，锚定后与名字脱钩；
- 定义不写凭证，「名 → 号」的绑定由登记处持有；
- 步骤凭证按「工作流凭证 + 步骤名」分层派生；
- 工作任务封面所引 `workflow_id` 由账本方在开任务时查填，落笔即封，封面按开任务时的名与号双锚存续（见 [工作任务](./work-task.md)）；
- 改名是身份中性操作：经登记处改绑，凭证不动；旧名随旧号占坑，区内名字只增不减。

### 判据

判据挂在步骤上，答「怎么算完」。判据两字段：`executor` 与 `description`，`executor` 答谁判，取 `rule`、`agent` 或 `human`：

- `rule` 判据的判法由平台定义；
- `agent` 与 `human` 只写 `description`——前者是给智能体的判准，后者是留给人拍板的事项。

### 约束

- 字段取值之外一律拒绝：出现不认识的字段、缺必填字段、取值不在枚举内，均视为不合语法；

### 关联

工作流隶属工作区（见 [工作区](../place/workspace.md)），被工作任务按名引用（见 [工作任务](./work-task.md)）。

## 领域事件

### 事件

- 工作流已创建（`WorkflowCreated`）

### 约束

- 事件负载至少携带工作流名、工作区 `id` 与 `workflow_id`——名与号双锚，供下游投影、汇总与审计使用。

## API端点

工作流建模为 REST 资源，端点挂载于工作区之下，路径里的 `{workflow}` 即工作流名。

### 工作流资源端点

- `POST /workbenches/{workbench}/workspaces/{workspace}/workflows`：创建工作流。请求自带定义与凭证，缺凭证即拒；服务端只验 UUID 格式与主键判重，不造号、不重算。
- `GET /workbenches/{workbench}/workspaces/{workspace}/workflows`：列出工作区下的工作流。
- `GET /workbenches/{workbench}/workspaces/{workspace}/workflows/{workflow}`：读取单个工作流。

### 子资源端点

步骤与判据内嵌在定义里，不设读写端点；名字的改绑单设一个：

- `PUT /workbenches/{workbench}/workspaces/{workspace}/workflows/{workflow}/name`：改名——凭证不变，新名区内唯一；旧名随旧号占坑。

### 约束

- 定义内容不设修改口，不设 DELETE：步骤与判据的修改不经服务端——服务端只收新定义、只读存量；名字是例外，走改名端点改绑，凭证不变；
- 创建工作流撞上区内同名即拒绝，不覆盖；
- 读取单个工作流的响应携带登记的 `workflow_id` 与各步骤凭证，供开任务引用与跨端对账；
- 无状态、幂等键等 API 口径见 [工作区](../place/workspace.md)。
