# 计费概览页

## 1. 基本信息

| 字段 | 内容 |
|------|------|
| 功能名称 ⭐ | 计费概览页 |
| 任务编号 | T019 |
| 所属模块 ⭐ | M6 商业化 |
| 优先级 | P1 |
| 指派给 | @frontend-dev |
| 预计工期 | 3 人日 |
| 创建日期 | 2026-04-23 |
| 目标上线日期 | 2026-05-15 |

## 2. 需求背景

### 2.1 问题描述
当前项目缺乏计费信息展示页面，用户需要：
- 查看当前套餐
- 查看配额使用情况
- 查看用量统计

### 2.2 目标用户
- 终端用户：查看计费信息
- 管理员：监控用量

### 2.3 预期效果
- 提供清晰的计费概览
- 支持配额使用可视化
- 支持用量趋势展示

## 3. 页面位置 ⭐

### 3.1 用户端
| 页面 | 路由 | 入口 |
|------|------|------|
| 计费概览 | `/billing` | 主导航「计费」 |

### 3.2 管理端（如适用）
无

### 3.3 页面原型/设计稿
- 参考：`packages/pages/billing/BillingOverview`

## 4. 功能范围

### 4.1 包含功能
- [ ] 当前套餐展示
- [ ] 配额使用情况
- [ ] 用量统计（按能力/按时间）
- [ ] 用量趋势图
- [ ] 升级入口
- [ ] 空态/加载态/错误态

### 4.2 不包含功能（明确排除）
- 套餐购买（由 T020 实现）
- 账单下载（由 T021 实现）
- 发票申请（由 T021 实现）

## 5. 数据结构 ⭐

### 5.1 新增/修改的数据模型

```typescript
// 当前套餐
export interface CurrentPlan {
  id: string;
  name: string;
  description: string;
  price: number;
  currency: string;
  quota: number;
  usedQuota: number;
  remainingQuota: number;
  usagePercentage: number;
  validUntil: string;
  features: string[];
  isAutoRenew: boolean;
}

// 用量统计
export interface UsageStats {
  totalCalls: number;
  totalCost: number;
  averageLatency: number;
  successRate: number;
  period: 'daily' | 'weekly' | 'monthly';
  startDate: string;
  endDate: string;
}

// 用量明细
export interface UsageDetail {
  abilityId: string;
  abilityName: string;
  callCount: number;
  cost: number;
  averageLatency: number;
  successRate: number;
}

// 用量趋势
export interface UsageTrend {
  date: string;
  calls: number;
  cost: number;
  successRate: number;
}
```

### 5.2 数据库变更（如适用）
无

### 5.3 类型定义位置
- 文件：`packages/shared/src/types/index.ts`
- 命名：`CurrentPlan`, `UsageStats`, `UsageDetail`, `UsageTrend`

## 6. 交互流程

### 6.1 主流程
```
[用户进入计费页] ──► [加载套餐信息] ──► [加载用量统计] ──► [展示数据]
```

### 6.2 异常流程
| 场景 | 触发条件 | 系统行为 | 用户反馈 |
|------|----------|----------|----------|
| 加载失败 | 网络错误 | 显示错误态 | 提示刷新 |
| 无套餐 | 用户未购买套餐 | 显示空态 | 提示购买 |
| 配额不足 | 用量超过 80% | 警告提示 | 提示升级 |

### 6.3 边界情况
- 大量用量数据：支持分页
- 多币种：支持货币切换
- 暗色模式：自动适配

## 7. 依赖组件 ⭐

### 7.1 基础组件（base/）
| 组件 | 用途 | 是否已存在 |
|------|------|-----------|
| Button | 操作按钮 | 是（T002） |
| Card | 信息卡片 | 是（T002） |
| Progress | 进度条 | 是（T002） |
| Badge | 状态标识 | 是（T002） |
| Tabs | 内容切换 | 是（T002） |
| Skeleton | 加载骨架 | 是（T002） |
| Empty | 空态展示 | 是（T002） |

### 7.2 业务组件（business/）
| 组件 | 用途 | 是否已存在 |
|------|------|-----------|
| BillingCard | 计费卡片 | 否 |
| UsageChart | 用量图表 | 否 |

### 7.3 页面块组件（blocks/）
| 组件 | 用途 | 是否已存在 |
|------|------|-----------|
| PlanInfo | 套餐信息区 | 否 |
| QuotaUsage | 配额使用区 | 否 |
| UsageStatistics | 用量统计区 | 否 |
| TrendChart | 趋势图区 | 否 |

### 7.4 页面模板（layout/）
| 模板 | 用途 |
|------|------|
| DetailPageShell | 详情页骨架 |

### 7.5 需新建组件
| 组件 | 所属分层 | 说明 |
|------|----------|------|
| BillingCard | business | 计费卡片 |
| UsageChart | business | 用量图表 |
| PlanInfo | blocks | 套餐信息区 |
| QuotaUsage | blocks | 配额使用区 |
| UsageStatistics | blocks | 用量统计区 |
| TrendChart | blocks | 趋势图区 |

## 8. API 接口 ⭐

### 8.1 新增接口

#### 当前套餐查询
- **Method**: GET
- **Path**: `/api/v1/billing/plan`
- **响应类型**: `CurrentPlan`
- **权限**: 用户（已登录）

**响应示例**:
```json
{
  "success": true,
  "data": {
    "id": "pro",
    "name": "专业版",
    "description": "适合小型团队",
    "price": 99,
    "currency": "CNY",
    "quota": 10000,
    "usedQuota": 6500,
    "remainingQuota": 3500,
    "usagePercentage": 65,
    "validUntil": "2027-04-24T00:00:00Z",
    "features": ["高级调用", "优先支持", "SLA 保障"],
    "isAutoRenew": true
  }
}
```

#### 用量统计查询
- **Method**: GET
- **Path**: `/api/v1/billing/usage`
- **请求类型**: `{ period: 'daily' | 'weekly' | 'monthly'; startDate?: string; endDate?: string }`
- **响应类型**: `{ stats: UsageStats; details: UsageDetail[]; trends: UsageTrend[] }`
- **权限**: 用户（已登录）

**响应示例**:
```json
{
  "success": true,
  "data": {
    "stats": {
      "totalCalls": 6500,
      "totalCost": 65,
      "averageLatency": 1200,
      "successRate": 98.5,
      "period": "monthly",
      "startDate": "2026-04-01",
      "endDate": "2026-04-30"
    },
    "details": [
      {
        "abilityId": "text-generation",
        "abilityName": "文本生成",
        "callCount": 3000,
        "cost": 30,
        "averageLatency": 1000,
        "successRate": 99
      }
    ],
    "trends": [
      { "date": "2026-04-01", "calls": 200, "cost": 2, "successRate": 98 },
      { "date": "2026-04-02", "calls": 250, "cost": 2.5, "successRate": 99 }
    ]
  }
}
```

### 8.2 修改接口
无

### 8.3 接口定义位置
- 文件：`packages/shared/src/api/services.ts`
- 命名：`userApi.billing.plan`, `userApi.billing.usage`

## 9. 验收标准 ⭐

### 9.1 功能验收
- [ ] 当前套餐展示正常
- [ ] 配额使用情况展示正常
- [ ] 用量统计展示正常
- [ ] 用量趋势图展示正常
- [ ] 升级入口可点击
- [ ] 配额不足警告正常
- [ ] 空态/加载态/错误态完整

### 9.2 技术验收
- [ ] TypeScript 类型完整，无 `any`
- [ ] 使用 Design Tokens，无散落样式
- [ ] 使用页面模板骨架
- [ ] 页面状态机完整（loading/empty/error/idle）
- [ ] 组件复用符合分层规范
- [ ] API 错误处理完整

### 9.3 性能验收
- [ ] 首屏加载 < 2s
- [ ] 图表渲染 < 500ms
- [ ] 无内存泄漏（useEffect 清理）

### 9.4 兼容性验收
- [ ] 桌面端 ≥1200px 正常显示
- [ ] 用户端移动端核心功能可用

## 10. 风险评估

### 10.1 是否影响现网 ⭐
- [x] **否** — 全新功能，不影响现有页面

### 10.2 是否依赖其他任务先完成 ⭐
- [x] **是** — 依赖任务：
  - [x] `T003` — 业务组件库 — 待开发
  - [x] `T004` — 页面模板 — 待开发

### 10.3 技术风险
| 风险 | 可能性 | 影响 | 应对措施 |
|------|--------|------|----------|
| 图表性能 | 中 | 中 | 数据聚合，限制点数 |
| 数据量大 | 中 | 中 | 分页加载 |
| 多币种 | 低 | 低 | 统一币种展示 |

## 11. 开发检查清单

### 11.1 开发前
- [ ] 需求已评审并确认
- [ ] 设计稿已确认
- [ ] 数据结构和 API 已评审
- [ ] 依赖任务已完成

### 11.2 开发中
- [ ] 遵循 `docs/dev-rules.md` 规范
- [ ] 遵循 `AI_RULES.md` 规范
- [ ] 自测覆盖所有验收标准

### 11.3 开发后
- [ ] 代码已提交 PR
- [ ] PR 已通过 Code Review
- [ ] 已联调通过
- [ ] 已更新文档（如需要）

## 12. 备注

- 用量趋势图支持日/周/月维度
- 配额不足 80% 时显示警告
- 支持暗色模式

---

**任务状态**：待评审 → 评审通过 → 开发中 → 自测完成 → CR 中 → 联调中 → 已上线
