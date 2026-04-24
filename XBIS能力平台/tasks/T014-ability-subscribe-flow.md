# 能力接入流程

## 1. 基本信息

| 字段 | 内容 |
|------|------|
| 功能名称 ⭐ | 能力接入流程 |
| 任务编号 | T014 |
| 所属模块 ⭐ | M4 能力平台 |
| 优先级 | P1 |
| 指派给 | @frontend-dev |
| 预计工期 | 3 人日 |
| 创建日期 | 2026-04-23 |
| 目标上线日期 | 2026-05-15 |

## 2. 需求背景

### 2.1 问题描述
当前项目缺乏标准化的能力接入流程，需要：
- 清晰的接入引导
- 套餐选择
- 密钥获取

### 2.2 目标用户
- 终端用户：接入能力使用
- 开发者：获取 API 密钥

### 2.3 预期效果
- 提供标准化的接入流程
- 支持套餐选择
- 支持密钥获取和管理

## 3. 页面位置 ⭐

### 3.1 用户端
| 页面 | 路由 | 入口 |
|------|------|------|
| 能力接入 | `/abilities/:id/subscribe` | 能力详情页「接入」按钮 |

### 3.2 管理端（如适用）
无

### 3.3 页面原型/设计稿
- 参考：`packages/pages/ability/AbilitySubscribeFlow`

## 4. 功能范围

### 4.1 包含功能
- [ ] 套餐选择
- [ ] 费用确认
- [ ] 接入确认
- [ ] 密钥展示
- [ ] 接入文档
- [ ] 空态/加载态/错误态

### 4.2 不包含功能（明确排除）
- 支付流程（由 T020 实现）
- 发票申请（由 T021 实现）
- 套餐管理（管理端功能）

## 5. 数据结构 ⭐

### 5.1 新增/修改的数据模型

```typescript
// 套餐选项
export interface PlanOption {
  id: string;
  name: string;
  description: string;
  price: number;
  currency: string;
  quota: number;
  overagePrice?: number;
  features: string[];
  isRecommended: boolean;
}

// 接入请求
export interface AbilitySubscribeRequest {
  abilityId: string;
  planId: string;
  couponCode?: string;
}

// 接入响应
export interface AbilitySubscribeResponse {
  subscriptionId: string;
  abilityId: string;
  planId: string;
  status: 'active' | 'pending' | 'expired';
  apiKey: string;
  apiSecret: string;
  quota: number;
  usedQuota: number;
  validUntil: string;
  createdAt: string;
}

// 接入信息
export interface SubscriptionInfo {
  subscriptionId: string;
  ability: AbilityCardItem;
  plan: PlanOption;
  status: 'active' | 'pending' | 'expired';
  quota: number;
  usedQuota: number;
  validUntil: string;
  createdAt: string;
}
```

### 5.2 数据库变更（如适用）
无

### 5.3 类型定义位置
- 文件：`packages/shared/src/types/index.ts`
- 命名：`PlanOption`, `AbilitySubscribeRequest`, `AbilitySubscribeResponse`, `SubscriptionInfo`

## 6. 交互流程

### 6.1 主流程
```
[用户点击接入] ──► [选择套餐] ──► [确认费用] ──► [确认接入] ──► [展示密钥] ──► [查看文档]
```

### 6.2 异常流程
| 场景 | 触发条件 | 系统行为 | 用户反馈 |
|------|----------|----------|----------|
| 加载失败 | 网络错误 | 显示错误态 | 提示刷新 |
| 已接入 | 用户已接入该能力 | 提示已接入 | 展示已有密钥 |
| 套餐不可用 | 套餐已下架 | 标记不可用 | 提示选择其他套餐 |
| 配额不足 | 免费配额已用完 | 提示升级 | 引导选择付费套餐 |

### 6.3 边界情况
- 密钥安全：仅展示一次，支持重新生成
- 套餐对比：支持多套餐对比
- 暗色模式：自动适配

## 7. 依赖组件 ⭐

### 7.1 基础组件（base/）
| 组件 | 用途 | 是否已存在 |
|------|------|-----------|
| Button | 操作按钮 | 是（T002） |
| Card | 套餐卡片 | 是（T002） |
| Radio | 单选 | 是（T002） |
| Modal | 确认弹窗 | 是（T002） |
| Steps | 步骤条 | 是（T002） |
| Skeleton | 加载骨架 | 是（T002） |
| Empty | 空态展示 | 是（T002） |

### 7.2 业务组件（business/）
| 组件 | 用途 | 是否已存在 |
|------|------|-----------|
| PlanCard | 套餐卡片 | 否 |
| ApiKeyDisplay | 密钥展示 | 否 |

### 7.3 页面块组件（blocks/）
| 组件 | 用途 | 是否已存在 |
|------|------|-----------|
| PlanSelector | 套餐选择区 | 否 |
| ConfirmPanel | 确认面板 | 否 |
| KeyDisplay | 密钥展示区 | 否 |
| DocPreview | 文档预览区 | 否 |

### 7.4 页面模板（layout/）
| 模板 | 用途 |
|------|------|
| FormPageShell | 表单页骨架 |

### 7.5 需新建组件
| 组件 | 所属分层 | 说明 |
|------|----------|------|
| PlanCard | business | 套餐卡片 |
| ApiKeyDisplay | business | 密钥展示组件 |
| PlanSelector | blocks | 套餐选择区 |
| ConfirmPanel | blocks | 确认面板 |
| KeyDisplay | blocks | 密钥展示区 |
| DocPreview | blocks | 文档预览区 |

## 8. API 接口 ⭐

### 8.1 新增接口

#### 获取套餐列表
- **Method**: GET
- **Path**: `/api/v1/abilities/:id/plans`
- **响应类型**: `PlanOption[]`
- **权限**: 用户（已登录）

**响应示例**:
```json
{
  "success": true,
  "data": [
    {
      "id": "free",
      "name": "免费版",
      "description": "适合个人开发者试用",
      "price": 0,
      "currency": "CNY",
      "quota": 100,
      "features": ["基础调用", "社区支持"],
      "isRecommended": false
    },
    {
      "id": "pro",
      "name": "专业版",
      "description": "适合小型团队",
      "price": 99,
      "currency": "CNY",
      "quota": 10000,
      "overagePrice": 0.01,
      "features": ["高级调用", "优先支持", " SLA 保障"],
      "isRecommended": true
    }
  ]
}
```

#### 接入能力
- **Method**: POST
- **Path**: `/api/v1/abilities/:id/subscribe`
- **请求类型**: `AbilitySubscribeRequest`
- **响应类型**: `AbilitySubscribeResponse`
- **权限**: 用户（已登录）

**请求示例**:
```json
{
  "abilityId": "text-generation",
  "planId": "pro",
  "couponCode": "SAVE20"
}
```

**响应示例**:
```json
{
  "success": true,
  "data": {
    "subscriptionId": "sub-001",
    "abilityId": "text-generation",
    "planId": "pro",
    "status": "active",
    "apiKey": "ak-xxxxxxxxxxxx",
    "apiSecret": "sk-xxxxxxxxxxxx",
    "quota": 10000,
    "usedQuota": 0,
    "validUntil": "2027-04-24T00:00:00Z",
    "createdAt": "2026-04-24T10:30:00Z"
  }
}
```

#### 重新生成密钥
- **Method**: POST
- **Path**: `/api/v1/subscriptions/:id/regenerate-key`
- **响应类型**: `{ apiKey: string; apiSecret: string }`
- **权限**: 用户（已登录）

### 8.2 修改接口
无

### 8.3 接口定义位置
- 文件：`packages/shared/src/api/services.ts`
- 命名：`userApi.abilities.plans`, `userApi.abilities.subscribe`, `userApi.subscriptions.regenerateKey`

## 9. 验收标准 ⭐

### 9.1 功能验收
- [ ] 套餐列表展示正常
- [ ] 套餐选择正常
- [ ] 费用确认正常
- [ ] 接入确认正常
- [ ] 密钥展示正常（仅展示一次）
- [ ] 支持重新生成密钥
- [ ] 接入文档展示正常
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
- [ ] 接入请求 < 500ms
- [ ] 无内存泄漏（useEffect 清理）

### 9.4 兼容性验收
- [ ] 桌面端 ≥1200px 正常显示
- [ ] 用户端移动端核心功能可用

## 10. 风险评估

### 10.1 是否影响现网 ⭐
- [x] **否** — 全新功能，不影响现有页面

### 10.2 是否依赖其他任务先完成 ⭐
- [x] **是** — 依赖任务：
  - [x] `T012` — 能力详情页 — 待开发

### 10.3 技术风险
| 风险 | 可能性 | 影响 | 应对措施 |
|------|--------|------|----------|
| 密钥泄露 | 低 | 高 | 仅展示一次，支持重新生成 |
| 套餐变更 | 中 | 中 | 接入时锁定套餐版本 |
| 并发接入 | 低 | 中 | 幂等处理 |

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

- 密钥仅展示一次，需提示用户保存
- 支持重新生成密钥，旧密钥失效
- 接入文档支持 Markdown 渲染

---

**任务状态**：待评审 → 评审通过 → 开发中 → 自测完成 → CR 中 → 联调中 → 已上线
