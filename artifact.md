# 工作物规格

本文定义工作物的通用语言、领域模型、领域事件与 API 端点。工作物的操作规范见知识工作手册《[工作物](../handbook/artifacts/index.md)》。

## 通用语言

| 术语 | 英文 | 定义 |
|------|------|------|
| 工作物 | Artifact | 知识工作的标准文档件，在流动链中承担一个明确角色 |
| 角色 | Role | 工作物在流动链中的职能：捕获入口、沉淀层、集合载体 |
| 规格 | Specification | 工作物的维护方式说明，按目标/流程/验收三段编写 |
| 条目 | Item | 工作物中的一条内容，流动的最小单元 |
| 流动 | Flow | 条目从上游工作物经加工向下游转出的过程 |
| 去处 | Destination | 条目一次流动的唯一目标：下游工作物或领域资产 |
| 流动记录 | Flow Record | 一次流动的追溯凭据：来源、去处、时间 |
| 发布 | Release | 手册规格变更的版本化发布：CHANGELOG 条目 + 版本标签 |

## 领域模型

### 聚合：Artifact（工作物）

| 字段 | 类型 | 说明 |
|------|------|------|
| id | string | 标识（slug，如 `journal`、`profile`、`handbook`） |
| title | string | 名称 |
| role | Role | 流动链角色，一件工作物一个角色（不变量） |
| specification | Specification | 规格：goal / process / acceptance 三段 |

**值对象 Role**：`capture-entry`（捕获入口）｜ `sediment-layer`（沉淀层）｜ `collection-carrier`（集合载体）

**值对象 Specification**：`goal`（目标）、`process`（流程步骤）、`acceptance`（验收标准）

### 实体：Item（条目）

| 字段 | 类型 | 说明 |
|------|------|------|
| id | string | 标识 |
| artifactId | string | 所属工作物 |
| content | string | 正文 |
| status | ItemStatus | 状态 |

**值对象 ItemStatus**：`captured`（已捕获）→ `organized`（已整理）→ `transferred`（已转出）

### 实体：FlowRecord（流动记录）

| 字段 | 类型 | 说明 |
|------|------|------|
| id | string | 标识 |
| itemId | string | 条目 |
| fromArtifactId | string | 来源工作物 |
| destination | Destination | 去处 |
| transferredAt | timestamp | 流动时间 |

**值对象 Destination**：`{ type: artifact, artifactId }` 或 `{ type: asset, domain, asset }`（对应领域的对应资产）

### 聚合：Release（发布，仅集合载体）

| 字段 | 类型 | 说明 |
|------|------|------|
| id | string | 标识 |
| artifactId | string | 所属工作物（手册） |
| version | string | 语义化版本 |
| changelog | string | 变更记录 |

### 不变量

1. **角色唯一**：一件 Artifact 只有一个 Role。
2. **单一去处**：一次流动只有一个 Destination。
3. **不留副本**：条目转出后，源工作物不再保留其原形式。
4. **可再找到**：每次流动生成 FlowRecord，去处必须存在且可检索。
5. **只向下游**：流动沿捕获入口 → 沉淀层 → 集合载体方向进行，不回流。

## 领域事件

| 事件 | 说明 | 载荷 |
|------|------|------|
| ArtifactRegistered | 工作物已注册 | artifactId, role, specification |
| SpecificationRevised | 规格已修订 | artifactId, specification |
| ItemCaptured | 条目已捕获 | itemId, artifactId, content |
| ItemOrganized | 条目已整理 | itemId, content |
| ItemTransferredOut | 条目已转出 | itemId, fromArtifactId, destination（触发源工作物移除原形式） |
| ReleasePublished | 版本已发布 | artifactId, version, changelog |

## API 端点

### 工作物

| 方法 | 路径 | 说明 |
|------|------|------|
| GET | `/artifacts` | 工作物列表（流动链视图） |
| POST | `/artifacts` | 注册工作物 |
| GET | `/artifacts/{artifactId}` | 工作物详情（角色与规格） |
| PATCH | `/artifacts/{artifactId}/specification` | 修订规格 |

### 条目与流动

| 方法 | 路径 | 说明 |
|------|------|------|
| POST | `/artifacts/{artifactId}/items` | 捕获条目 |
| GET | `/artifacts/{artifactId}/items` | 条目列表（可按 status 过滤） |
| GET | `/items/{itemId}` | 条目详情 |
| PATCH | `/items/{itemId}` | 整理条目 |
| POST | `/items/{itemId}/transfer` | 转出条目（请求体含 destination，转出后源工作物移除原形式） |
| GET | `/items/{itemId}/flow-records` | 条目的流动记录（可再找到） |

### 发布

| 方法 | 路径 | 说明 |
|------|------|------|
| GET | `/artifacts/{artifactId}/releases` | 发布列表 |
| POST | `/artifacts/{artifactId}/releases` | 发布版本（version + changelog，预检查通过后创建标签与 Release） |
