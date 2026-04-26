# T003 业务组件库 — 设计评审报告

> 评审日期: 2026-04-24  
> 评审人: 技术负责人 + 架构审查官  
> 状态: Passed with changes

---

## 评审结果

### 状态: 👉 Passed with changes

---

## 问题列表

### 问题1: 20 个组件一次性开发，粒度控制风险
**严重程度: 中**

20 个业务组件一次性开发，存在过度设计风险。部分组件可能使用频率极低（如 AiSparkle、CouponTag）。

### 问题2: UsageChart 依赖图表库未声明
**严重程度: 中**

UsageChart 组件需要图表库支持（如 ECharts、Recharts 或 Ant Design Charts），但任务卡和设计方案均未声明依赖库。

### 问题3: DataMetric 和 PriceDisplay 为纯样式组件，是否需独立
**严重程度: 低**

DataMetric 和 PriceDisplay 为纯样式组件，功能简单，可能不需要独立为业务组件。

### 问题4: 与现有页面内嵌组件重复风险高
**严重程度: 高**

现有页面（如 ApiMarket、Invocations）可能已有类似实现，直接创建新组件会导致重复。

### 问题5: 类型依赖使用旧命名（ApiMarketItem）
**严重程度: 中**

AbilityCard 依赖类型 `ApiMarketItem`，但项目正在迁移到 `Ability` 类型，应使用新命名。

---

## 修改建议

### 建议1: 分优先级分批开发（必须修改）
按使用频率分批次：
- **第一批（P0）**: AbilityCard, TaskItem, TaskStatusBadge, PlanCard, ApiKeyItem
- **第二批（P1）**: BillingCard, UsageChart, QuotaBar, RiskBadge, ExecutionModeTag
- **第三批（P2）**: 其余组件按需开发

### 建议2: 明确图表库依赖（必须修改）
在任务卡中明确声明图表库选择（推荐 Recharts 或 ECharts），并说明理由。

### 建议3: 合并纯样式组件（建议修改）
将 DataMetric、PriceDisplay、AiSparkle 合并为 `Display` 组件或降级为工具函数。

### 建议4: 先梳理现有组件再开发（必须修改）
开发前必须：
1. 梳理现有页面中的可复用组件
2. 标记可提取的组件
3. 避免重复实现

### 建议5: 统一使用新类型命名（必须修改）
所有类型引用统一使用新命名：
- `ApiMarketItem` → `Ability`
- `Invocation` → `Job`
- `ApiKey` → `AccessKey`

---

## 是否允许进入开发

### 👉 YES（附条件通过）

**必须完成的条件**:
1. 分优先级分批开发（建议1）
2. 明确图表库依赖（建议2）
3. 先梳理现有组件（建议4）
4. 统一类型命名（建议5）

---

## 评审总结

| 检查项 | 结果 | 说明 |
|--------|------|------|
| 组件复用原则 | ⚠️ | 需先梳理现有组件 |
| 页面模板使用 | N/A | 无页面 |
| 重复组件 | ⚠️ | 与现有页面组件重复风险高 |
| 分层规范 | ✅ | 符合 base → business |
| API 合理性 | N/A | 无 API |
| 状态设计完整性 | ✅ | 纯展示组件 |
| 能力平台架构 | ⚠️ | 使用旧类型命名 |
| 过度设计 | ⚠️ | 20 个组件一次性开发 |
| 未声明数据模型 | ⚠️ | 图表库依赖未声明 |

**风险等级**: 中
