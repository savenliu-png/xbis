# 管理端套餐配置

## 1. 基本信息

| 字段 | 内容 |
|------|------|
| 功能名称 ⭐ | 管理端套餐配置 |
| 任务编号 | T031 |
| 所属模块 ⭐ | M14 计费与套餐 |
| 优先级 | P1 |
| 指派给 | @frontend-dev |
| 预计工期 | 3 人日 |
| 创建日期 | 2026-04-23 |
| 目标上线日期 | 2026-05-22 |

## 2. 需求背景

### 2.1 问题描述
当前项目缺乏管理端套餐配置页面，管理员需要：
- 创建和编辑套餐
- 配置计费规则
- 配置配额

### 2.2 目标用户
- 管理员：配置套餐
- 运营人员：调整价格

### 2.3 预期效果
- 提供完整的套餐配置
- 支持灵活的计费规则
- 支持配额配置

## 3. 页面位置 ⭐

### 3.1 用户端
无

### 3.2 管理端
| 页面 | 路由 | 入口 |
|------|------|------|
| 套餐配置 | `/admin/billing/plans` | 管理端导航「套餐配置」 |

### 3.3 页面原型/设计稿
- 参考：`packages/pages/admin/BillingConfig`

## 4. 功能范围

### 4.1 包含功能
- [ ] 套餐列表展示
- [ ] 套餐创建
- [ ] 套餐编辑
- [ ] 套餐删除
- [ ] 计费规则配置
- [ ] 配额配置
- [ ] 套餐状态管理
- [ ] 空态/加载态/错误态

### 4.2 不包含功能（明确排除）
- 优惠券管理（V2 迭代）
- 促销活动（V2 迭代）
- 退款规则（V2 迭代）

## 5. 数据结构 ⭐

### 5.1 新增/修改的数据模型

```typescript
// 管理端套餐项
export interface AdminPlanItem {
  id: string;
  name: string;
  description: string;
  price: number;
  originalPrice?: number;
  currency: string;
  quota: number;
  overagePrice?: number;
  features: PlanFeature[];
  isActive: boolean;
  isPublic: boolean;
  sortOrder: number;
  createdAt: string;
  updatedAt: string;
}

// 套餐表单
export interface PlanForm {
  name: string;
  description: string;
  price: number;
  originalPrice?: number;
  currency: string;
  quota: number;
  overagePrice?: number;
  features: PlanFeature[];
  billingRules: BillingRule[];
  isActive: boolean;
  isPublic: boolean;
}

// 计费规则
export interface BillingRule {
  type: 'per_call' | 'per_token' | 'per_minute';
  unitPrice: number;
  freeQuota?: number;
  tieredPricing?: TieredPrice[];
}

// 阶梯价格
export interface TieredPrice {
  min: number;
  max?: number;
  price: number;
}
```

### 5.2 数据库变更（如适用）
无

### 5.3 类型定义位置
- 文件：`packages/shared/src/types/index.ts`
- 命名：`AdminPlanItem`, `PlanForm`, `BillingRule`, `TieredPrice`

## 6. 交互流程

### 6.1 主流程
```
[管理员进入套餐配置] ──► [加载套餐列表] ──► [展示数据] ──► [管理员创建/编辑/删除]
```

### 6.2 异常流程
| 场景 | 触发条件 | 系统行为 | 用户反馈 |
|------|----------|----------|----------|
| 加载失败 | 网络错误 | 显示错误态 | 提示刷新 |
| 无套餐 | 无套餐 | 显示空态 | 提示创建 |
| 删除失败 | 套餐有用户 | 返回错误 | 提示先下架 |
| 保存失败 | 数据无效 | 返回错误 | 提示修改 |

### 6.3 边界情况
- 多币种：支持货币选择
- 阶梯价格：支持多阶梯
- 暗色模式：自动适配

## 7. 依赖组件 ⭐

### 7.1 基础组件（base/）
| 组件 | 用途 | 是否已存在 |
|------|------|-----------|
| Button | 操作按钮 | 是（T002） |
| Input | 文本输入 | 是（T002） |
| Table | 套餐表格 | 是（T002） |
| Tag | 状态标签 | 是（T002） |
| Badge | 状态标识 | 是（T002） |
| Switch | 状态切换 | 是（T002） |
| Modal | 确认弹窗 | 是（T002） |
| Form | 表单容器 | 是（T002） |
| Select | 下拉选择 | 是（T002） |
| Skeleton | 加载骨架 | 是（T002） |
| Empty | 空态展示 | 是（T002） |
| Pagination | 分页 | 是（T002） |

### 7.2 业务组件（business/）
| 组件 | 用途 | 是否已存在 |
|------|------|-----------|
| PlanCard | 套餐卡片 | 是（T014） |

### 7.3 页面块组件（blocks/）
| 组件 | 用途 | 是否已存在 |
|------|------|-----------|
| PlanTable | 套餐表格 | 否 |
| PlanForm | 套餐表单 | 否 |
| BillingRuleEditor | 计费规则编辑器 | 否 |

### 7.4 页面模板（layout/）
| 模板 | 用途 |
|------|------|
| AdminPageShell | 管理页骨架 |

### 7.5 需新建组件
| 组件 | 所属分层 | 说明 |
|------|----------|------|
| PlanTable | blocks | 套餐表格 |
| PlanForm | blocks | 套餐表单 |
| BillingRuleEditor | blocks | 计费规则编辑器 |

## 8. API 接口 ⭐

### 8.1 新增接口

#### 套餐列表查询（管理端）
- **Method**: GET
- **Path**: `/admin-api/v1/billing/plans`
- **响应类型**: `{ items: AdminPlanItem[]; total: number }`
- **权限**: 管理员

**响应示例**:
```json
{
  "success": true,
  "data": {
    "items": [
      {
        "id": "pro",
        "name": "专业版",
        "description": "适合小型团队",
        "price": 99,
        "originalPrice": 129,
        "currency": "CNY",
        "quota": 10000,
        "overagePrice": 0.01,
        "features": [
          { "name": "基础调用", "included": true },
          { "name": "优先支持", "included": true }
        ],
        "isActive": true,
        "isPublic": true,
        "sortOrder": 1,
        "createdAt": "2026-01-01T00:00:00Z",
        "updatedAt": "2026-04-24T00:00:00Z"
      }
    ],
    "total": 5
  }
}
```

#### 套餐创建
- **Method**: POST
- **Path**: `/admin-api/v1/billing/plans`
- **请求类型**: `PlanForm`
- **响应类型**: `AdminPlanItem`
- **权限**: 管理员

#### 套餐更新
- **Method**: PUT
- **Path**: `/admin-api/v1/billing/plans/:id`
- **请求类型**: `PlanForm`
- **响应类型**: `AdminPlanItem`
- **权限**: 管理员

#### 套餐删除
- **Method**: DELETE
- **Path**: `/admin-api/v1/billing/plans/:id`
- **权限**: 管理员

### 8.2 修改接口
无

### 8.3 接口定义位置
- 文件：`packages/shared/src/api/services.ts`
- 命名：`adminApi.billing.plans`, `adminApi.billing.createPlan`, `adminApi.billing.updatePlan`, `adminApi.billing.deletePlan`

## 9. 验收标准 ⭐

### 9.1 功能验收
- [ ] 套餐列表展示正常
- [ ] 套餐创建正常
- [ ] 套餐编辑正常
- [ ] 套餐删除正常
- [ ] 计费规则配置正常
- [ ] 配额配置正常
- [ ] 套餐状态管理正常
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
- [ ] 保存响应 < 500ms
- [ ] 无内存泄漏（useEffect 清理）

### 9.4 兼容性验收
- [ ] 桌面端 ≥1200px 正常显示
- [ ] 管理端无需移动端设计

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
| 计费规则复杂 | 中 | 中 | 简化配置 |
| 误删除 | 中 | 高 | 二次确认 |
| 价格错误 | 中 | 高 | 预览确认 |

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

- 套餐删除需二次确认
- 计费规则支持阶梯定价
- 支持暗色模式

---

**任务状态**：待评审 → 评审通过 → 开发中 → 自测完成 → CR 中 → 联调中 → 已上线
