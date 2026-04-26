# T005 ability 数据模型 — 工程级设计方案

> 任务卡: [T005-ability-data-model.md](../T005-ability-data-model.md)  
> 设计输入: docs/dev-rules.md, docs/component-architecture.md  
> 输出日期: 2026-04-24

---

## 1. 页面结构

本任务为**数据层**建设，无页面结构。输出物为 TypeScript 类型定义 + 数据库 Schema 文档。

```
packages/shared/src/types/
├── index.ts                        # 统一导出入口
├── ability.ts                      # ability 相关类型
├── job.ts                          # job 相关类型
├── executor.ts                     # executor 相关类型
├── billing.ts                      # 计费相关类型
├── user.ts                         # 用户相关类型
└── common.ts                       # 通用类型
```

---

## 2. 组件拆分

无 UI 组件。本任务输出为纯类型定义。

### 类型定义清单

| 类型 | 文件 | 说明 |
|------|------|------|
| Ability | `types/ability.ts` | 能力实体 |
| AbilityVersion | `types/ability.ts` | 能力版本 |
| AbilityUiSchema | `types/ability.ts` | UI Schema |
| AbilityListParams | `types/ability.ts` | 列表查询参数 |
| AbilityListResponse | `types/ability.ts` | 列表响应 |
| AbilityDetailResponse | `types/ability.ts` | 详情响应 |
| AbilityCreateRequest | `types/ability.ts` | 创建请求 |
| AbilityUpdateRequest | `types/ability.ts` | 更新请求 |

---

## 3. 数据流

无运行时数据流。类型定义为编译时静态检查。

```
API 契约 ──► TypeScript 类型 ──► 前端/后端共享 ──► 编译时检查
```

---

## 4. 状态管理

无状态管理需求。

---

## 5. API 调用

无 API 调用。本任务为类型定义。

---

## 6. 用户交互流程

无用户交互。

---

## 7. 边界与异常

| 场景 | 处理方案 |
|------|----------|
| 类型定义与后端不一致 | 建立类型同步机制，定期对齐 |
| 字段类型冲突 | 使用联合类型或泛型处理 |
| 可选字段缺失 | 使用 `?` 标记可选字段 |

---

## 8. 性能优化

- 类型定义仅在编译时使用，无运行时开销
- 使用 `interface` 而非 `type` 支持声明合并

---

## 9. 风险点

| 风险 | 等级 | 说明 | 应对 |
|------|------|------|------|
| 与后端 Schema 不一致 | 高 | 类型定义可能滞后于后端变更 | 建立类型同步机制 |
| 类型定义过于复杂 | 中 | 嵌套类型可能导致维护困难 | 拆分类型到独立文件 |
| 类型复用不足 | 低 | 相似类型可能重复定义 | 提取公共类型到 common.ts |

---

## 10. 开发步骤拆分

### Step 1: 核心类型定义（1 人日）
- [ ] Ability 实体类型
- [ ] AbilityVersion 类型
- [ ] AbilityUiSchema 类型

### Step 2: API 类型定义（0.5 人日）
- [ ] AbilityListParams 类型
- [ ] AbilityListResponse 类型
- [ ] AbilityDetailResponse 类型
- [ ] AbilityCreateRequest 类型
- [ ] AbilityUpdateRequest 类型

### Step 3: 数据库 Schema 文档（0.5 人日）
- [ ] ability 表 Schema
- [ ] ability_version 表 Schema
- [ ] ability_ui_schema 表 Schema

### Step 4: 类型导出 & 文档（0.5 人日）
- [ ] 创建 `index.ts` 统一导出
- [ ] 编写类型使用文档
