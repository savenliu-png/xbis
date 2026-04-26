# T003 业务组件库 — 修正版设计方案（V2）

> 原设计方案: [T003-business-components-spec.md](../T003-business-components-spec.md)
> 评审报告: [T003-business-components-Reviewer.md](../T003-business-components-Reviewer.md)
> 输出日期: 2026-04-24
> 修正版本: V2

---

## 一、最终评审状态

### 👉 Passed with changes

---

## 二、必须修改项（Blocking）

| 序号 | 问题 | 修复位置 |
|------|------|----------|
| 1 | 分优先级分批开发 | 第 10 节开发步骤 |
| 2 | 明确图表库依赖 | 第 2 节组件拆分 |
| 3 | 先梳理现有组件再开发 | 第 10 节开发步骤 |
| 4 | 统一使用新类型命名 | 第 2 节组件拆分 |

---

## 三、建议修改项（Optional）

| 序号 | 问题 | 修复位置 |
|------|------|----------|
| 1 | 合并纯样式组件（DataMetric/PriceDisplay/AiSparkle） | 第 2 节组件拆分 |

---

## 四、修正版设计方案（V2）

### 1. 页面结构

无页面结构，本任务为组件库建设。

---

### 2. 组件拆分

#### business/ 层组件（V2 分优先级）【已修复】

**第一批（P0 - 核心组件）**：

| 组件 | 依赖类型 | 说明 |
|------|----------|------|
| AbilityCard | `Ability` 【已修复：使用新类型命名】 | 能力卡片 |
| TaskItem | `Job` 【已修复】 | 任务列表项 |
| TaskStatusBadge | `JobStatus` 【已修复】 | 任务状态徽章 |
| PlanCard | `Plan` | 套餐卡片 |
| ApiKeyItem | `AccessKey` 【已修复】 | API Key 列表项 |

**第二批（P1 - 重要组件）**：

| 组件 | 依赖类型 | 说明 |
|------|----------|------|
| BillingCard | `BillingItem` | 账单卡片 |
| UsageChart | `UsageData` | 用量图表 |
| QuotaBar | `QuotaUsage` | 配额进度条 |
| RiskBadge | `RiskLevel` | 风险等级徽章 |
| ExecutionModeTag | `ExecutionMode` | 执行模式标签 |

**第三批（P2 - 按需开发）**：

| 组件 | 依赖类型 | 说明 |
|------|----------|------|
| InvoiceCard | `Invoice` | 发票卡片 |
| NotificationBell | `Notification` | 通知铃铛 |
| CopyButton | - | 复制按钮（从 base 层移至 business 层）【已修复】 |
| DataMetric | `MetricData` | 数据指标展示 |
| PriceDisplay | `PriceData` | 价格展示 |
| AiSparkle | - | AI 特性标识 |
| CouponTag | `Coupon` | 优惠券标签 |

#### 图表库选择【已修复】

推荐使用 **Recharts**，理由：
- React 原生支持，API 友好
- 体积小，Tree-shaking 友好
- 与 Ant Design 风格一致
- 支持响应式

#### 纯样式组件处理【建议修改】

- DataMetric、PriceDisplay、AiSparkle → 合并为 `Display` 组件或降级为工具函数

---

### 3. 数据流

无运行时数据流。

---

### 4. 状态管理

无状态管理需求。

---

### 5. API 调用

无 API 调用。

---

### 6. 用户交互流程

无用户交互流程。

---

### 7. 边界与异常

| 场景 | 处理方案 |
|------|----------|
| 类型不匹配 | 使用 TypeScript 严格模式校验 |

---

### 8. 性能优化

- 按需导出，支持 Tree-shaking
- 图表组件懒加载

---

### 9. 风险点

| 风险 | 等级 | 说明 | 应对 |
|------|------|------|------|
| 与现有页面组件重复 | 高 | 现有页面可能已有类似实现 | 开发前梳理现有组件，标记可提取的组件 |
| 图表库体积 | 中 | Recharts 可能增加包体积 | 按需引入，懒加载 |

---

### 10. 开发步骤拆分

#### Step 1: 梳理现有组件（0.5 人日）【已修复】
- [ ] 梳理现有页面中的可复用组件
- [ ] 标记可提取的组件
- [ ] 避免重复实现

#### Step 2: 第一批 P0 组件（1.5 人日）【已修复】
- [ ] AbilityCard
- [ ] TaskItem
- [ ] TaskStatusBadge
- [ ] PlanCard
- [ ] ApiKeyItem

#### Step 3: 第二批 P1 组件（1.5 人日）【已修复】
- [ ] BillingCard
- [ ] UsageChart（集成 Recharts）
- [ ] QuotaBar
- [ ] RiskBadge
- [ ] ExecutionModeTag

#### Step 4: 第三批 P2 组件（按需）【已修复】
- [ ] 其余组件按需开发

---

## 五、修改对比（Diff）

| 项目 | 原设计 | V2 修正 |
|------|--------|---------|
| **开发批次** | 20 个组件一次性开发 | 分 P0/P1/P2 三批开发 |
| **图表库** | 未声明 | 明确使用 Recharts |
| **类型命名** | `ApiMarketItem` / `Invocation` / `ApiKey` | `Ability` / `Job` / `AccessKey` |
| **现有组件梳理** | 未提及 | 开发前必须梳理现有组件 |
| **CopyButton** | 在 base 层 | 移至 business 层 |
| **纯样式组件** | 独立组件 | 建议合并为 Display 组件 |

---

## 六、是否允许进入开发

### 👉 YES

**原因**：
1. 所有阻塞项已修复（分优先级开发、明确图表库、梳理现有组件、统一类型命名）
2. 组件分层已调整
3. 风险等级：中（可控）
