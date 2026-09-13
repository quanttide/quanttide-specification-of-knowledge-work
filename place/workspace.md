# 工作区

工作区（Workspace）是一次工作的边界：材料、产物、工作流、任务都归属于它；区内工件与过程互相可见、可流转，区间彼此隔绝。

工作区是建模单位，只认内容、不认位置——工件在物理上从哪来、落在哪，由平台在装载时决定；同一个工作区，资源可以来自多处、任意组合。

## 领域模型

工作区包括以下字段：

- `id`(UUID): 唯一标识
- `name`(Slug): 工作区名称，用于编排、URL等
- `workflows`(List<Workflow>): 工作流列表
- `tasks`(List<Task>): 任务列表
- `artifacts`(List<Artifact>): 产物列表
- `materials`(List<Material>): 材料列表

## 领域事件



## API 规格
