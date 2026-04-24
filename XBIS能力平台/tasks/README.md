# XBIS 任务池（Task Pool）

> 基于《01 xbis-front 系统重构方案.md》
> 输出日期：2026-04-23
> 版本：v1.0

---

## 文件结构

```
tasks/
├── README.md                          # 本文件：任务池索引
├── system-modules.md                  # 系统模块划分（17 个模块）
├── sprint-backlog.md                  # Sprint Backlog（36 个工程任务）
├── task-dependencies.md               # 任务依赖关系（依赖链 + 并行组 + 阻塞任务）
├── development-sequence.md            # 推荐开发顺序（6 个 Step，10 周）
├── milestone-backlog.md               # Milestone Backlog（3 个版本规划）
├── risks-and-checklists.md            # 风险点汇总 + AI 强制自检 + 开发后自检
├── task-template.md                   # 标准任务卡模板（引用 docs/task-template.md）
├── feature-ability-center.md          # 示例任务卡：能力中心
├── feature-task-platform.md           # 示例任务卡：任务平台
├── feature-billing-upgrade.md         # 示例任务卡：计费系统升级
└── T001-T036/                         # 待创建：具体任务卡目录
```

---

## 快速导航

| 文档 | 用途 | 关键内容 |
|---|---|---|
| [system-modules.md](system-modules.md) | 系统模块划分 | 17 个模块，P0/P1/P2 分级 |
| [sprint-backlog.md](sprint-backlog.md) | 任务池总表 | 36 个任务，按优先级/端/模块统计 |
| [task-dependencies.md](task-dependencies.md) | 依赖关系 | 6 层依赖链，8 个并行组，阻塞任务 |
| [development-sequence.md](development-sequence.md) | 开发顺序 | 6 个 Step，10 周排期，人员配置 |
| [milestone-backlog.md](milestone-backlog.md) | 版本规划 | 3 个 Milestone，灰度策略 |
| [risks-and-checklists.md](risks-and-checklists.md) | 风险与自检 | 风险点汇总，AI 自检，Phase C 准入 |

---

## 核心数据

### 任务统计

| 维度 | 数量 |
|---|---|
| 总任务数 | 36 个 |
| P0 任务 | 14 个 |
| P1 任务 | 21 个 |
| P2 任务 | 1 个 |
| 前端任务 | 27 个（75%） |
| 后端任务 | 9 个（25%） |

### 模块覆盖

| 层级 | 模块数 | 模块 |
|---|---|---|
| 基础层 | 3 | M1 设计系统、M2 数据层、M3 API 层 |
| 用户端 | 5 | M4 能力平台、M5 任务系统、M6 商业化、M7 开发者中心、M8 个人中心 |
| 管理端 | 9 | M9-M17 概览/能力/执行器/任务/用户/计费/财务/内容/配置 |

### 时间线

| 阶段 | 周期 | 里程碑 |
|---|---|---|
| Step 1：基础层 | Week 1-2 | — |
| Step 2：API 层 | Week 3 | — |
| Step 3：能力平台 | Week 4 | — |
| Step 4：任务系统 | Week 5-6 | **Milestone 1（MVP）** |
| Step 5：增强 + 管理端 | Week 7-9 | **Milestone 2（增强版）** |
| Step 6：导航重构 | Week 10 | **Milestone 3（平台化）** |

---

## 使用流程

### 1. 查看任务池

```bash
# 查看所有任务
open tasks/sprint-backlog.md

# 查看依赖关系
open tasks/task-dependencies.md

# 查看开发顺序
open tasks/development-sequence.md
```

### 2. 创建具体任务卡

```bash
# 复制模板
cp docs/task-template.md tasks/T011-ability-list.md

# 填写任务信息
# 参考 sprint-backlog.md 中的任务定义
```

### 3. 跟踪任务状态

```bash
# 更新任务状态（在 sprint-backlog.md 中标记）
# 更新里程碑状态（在 milestone-backlog.md 中标记）
```

### 4. 开发后自检

```bash
# 参考 risks-and-checklists.md
# 完成 AI 强制自检
# 完成 Phase C 准入检查
```

---

## 关键约定

### 任务编号规则

| 编号格式 | 示例 | 说明 |
|---|---|---|
| T001-T099 | T011 | Sprint Backlog 任务 |
| FEATURE-YYYYMM-NNN | FEATURE-202604-001 | 功能特性任务 |

### 优先级定义

| 优先级 | 说明 | 响应时间 |
|---|---|---|
| P0 | 必须先做，否则系统不可运行 | 立即 |
| P1 | 增强体验，提升效率 | 1 周内 |
| P2 | 扩展能力，未来规划 | 1 月内 |

### 状态流转

```
待评审 → 评审通过 → 开发中 → 自测完成 → CR 中 → 联调中 → 已上线
```

---

## 相关文档

| 文档 | 路径 | 说明 |
|---|---|---|
| 系统重构方案 | `docs/方案&配置/01 xbis-front 系统重构方案.md` | 重构总体方案 |
| 设计 Tokens | `docs/design-tokens.md` | 设计系统规范 |
| 组件架构 | `docs/component-architecture.md` | 组件分层规范 |
| 页面模板 | `docs/page-templates.md` | 页面模板规范 |
| 开发规则 | `docs/dev-rules.md` | 开发规范 |
| AI 规则 | `AI_RULES.md` | AI 开发指南 |
| 任务模板 | `docs/task-template.md` | 标准任务卡模板 |

---

## 维护记录

| 日期 | 版本 | 更新内容 | 更新人 |
|---|---|---|---|
| 2026-04-23 | v1.0 | 初始版本，包含 36 个任务 | AI 助手 |

---

## 联系方式

如有问题或建议，请联系：
- 技术负责人：@frontend-dev
- 产品经理：@product-manager
- 项目经理：@project-manager
