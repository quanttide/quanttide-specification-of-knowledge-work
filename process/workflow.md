# 工作流

工作流（Workflow）是过程的编排定义：以有向无环图描述任务之间的衔接。

## 语法

工作流是一份 YAML，落在数据仓的 `workflows/<工作流>.yaml`，文件名即工作流名。

```yaml
name: 工作流名
description: 一句话说清这条工作流干什么
steps:
  - name: 步骤名
    description: 这一步做什么
    executor: agent
    criteria:
      - executor: rule
        path: data/journal/README.md
```

顶层三个字段：`name`、`description` 与 `steps`。`name` 即工作流名，与文件名一致。一条工作流至少一个步骤，步骤的顺序即衔接的顺序。

步骤四个字段：`name`（必填）、`description`、`executor`、`criteria`。`executor` 答谁做这一步，取 `agent` 或 `human`，缺省为 `agent`。

判据三个字段：`executor`（必填）、`description`、判法。`executor` 答谁判这一条，取 `rule`、`agent` 或 `human`。判据为 `rule` 时必须且只能写一种判法：`path` 取路径存在，`absent` 取路径不存在，`file` 与 `contains` 成对取文件含这段文字，`run` 取命令退出码为零；判据为 `agent` 或 `human` 时只写 `description`，前者是给智能体的判准，后者是留给人拍板的事项。

`description` 可省，省去时按判法生成一句。

字段取值之外一律拒绝：出现不认识的字段、缺必填字段、取值不在枚举内，均视为不合语法。

判据里的路径相对工作区根。`{{report}}`、`{{journal}}`、`{{log}}`、`{{artifacts}}` 在任务执行时换成本次任务的实际位置。
