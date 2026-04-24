# 账单与发票

## 1. 基本信息

| 字段 | 内容 |
|------|------|
| 功能名称 ⭐ | 账单与发票 |
| 任务编号 | T021 |
| 所属模块 ⭐ | M6 商业化 |
| 优先级 | P1 |
| 指派给 | @frontend-dev |
| 预计工期 | 3 人日 |
| 创建日期 | 2026-04-23 |
| 目标上线日期 | 2026-05-18 |

## 2. 需求背景

### 2.1 问题描述
当前项目缺乏账单和发票管理功能，用户需要：
- 查看历史账单
- 申请发票
- 下载账单和发票

### 2.2 目标用户
- 终端用户：查看账单和申请发票
- 财务人员：下载发票

### 2.3 预期效果
- 提供清晰的账单列表
- 支持发票申请
- 支持账单和发票下载

## 3. 页面位置 ⭐

### 3.1 用户端
| 页面 | 路由 | 入口 |
|------|------|------|
| 账单与发票 | `/billing/invoices` | 计费概览页「账单」入口 |

### 3.2 管理端（如适用）
无

### 3.3 页面原型/设计稿
- 参考：`packages/pages/billing/InvoicePage`

## 4. 功能范围

### 4.1 包含功能
- [ ] 账单列表展示
- [ ] 账单详情
- [ ] 发票申请
- [ ] 发票状态跟踪
- [ ] 账单下载（PDF）
- [ ] 发票下载（PDF）
- [ ] 空态/加载态/错误态

### 4.2 不包含功能（明确排除）
- 自动开票（V2 迭代）
- 发票邮寄（V2 迭代）
- 账单自动发送（V2 迭代）

## 5. 数据结构 ⭐

### 5.1 新增/修改的数据模型

```typescript
// 账单项
export interface InvoiceItem {
  id: string;
  invoiceId: string;
  orderId: string;
  planId: string;
  planName: string;
  amount: number;
  currency: string;
  status: 'pending' | 'paid' | 'failed' | 'refunded';
  paymentMethod?: string;
  paidAt?: string;
  createdAt: string;
  periodStart?: string;
  periodEnd?: string;
}

// 发票
export interface Invoice {
  id: string;
  invoiceId: string;
  invoiceNumber: string;
  invoiceType: 'personal' | 'enterprise';
  title: string;
  taxNumber?: string;
  amount: number;
  currency: string;
  status: 'pending' | 'processing' | 'completed' | 'failed';
  downloadUrl?: string;
  createdAt: string;
  completedAt?: string;
}

// 发票申请请求
export interface InvoiceApplyRequest {
  invoiceType: 'personal' | 'enterprise';
  title: string;
  taxNumber?: string;
  email: string;
  address?: string;
  phone?: string;
  bankName?: string;
  bankAccount?: string;
  orderIds: string[];
}

// 发票申请响应
export interface InvoiceApplyResponse {
  invoiceId: string;
  status: 'pending';
  estimatedCompletion: string;
}
```

### 5.2 数据库变更（如适用）
无

### 5.3 类型定义位置
- 文件：`packages/shared/src/types/index.ts`
- 命名：`InvoiceItem`, `Invoice`, `InvoiceApplyRequest`, `InvoiceApplyResponse`

## 6. 交互流程

### 6.1 主流程
```
[用户进入账单页] ──► [加载账单列表] ──► [选择账单] ──► [申请发票] ──► [填写信息] ──► [提交申请]
```

### 6.2 异常流程
| 场景 | 触发条件 | 系统行为 | 用户反馈 |
|------|----------|----------|----------|
| 加载失败 | 网络错误 | 显示错误态 | 提示刷新 |
| 无账单 | 无消费记录 | 显示空态 | 提示消费后查看 |
| 申请失败 | 信息不完整 | 返回错误 | 提示补充信息 |
| 下载失败 | 文件不存在 | 返回错误 | 提示联系客服 |

### 6.3 边界情况
- 大量账单：支持分页
- 多币种：支持货币切换
- 暗色模式：自动适配

## 7. 依赖组件 ⭐

### 7.1 基础组件（base/）
| 组件 | 用途 | 是否已存在 |
|------|------|-----------|
| Button | 操作按钮 | 是（T002） |
| Card | 信息卡片 | 是（T002） |
| Table | 账单表格 | 是（T002） |
| Modal | 申请弹窗 | 是（T002） |
| Form | 表单容器 | 是（T002） |
| Input | 文本输入 | 是（T002） |
| Select | 下拉选择 | 是（T002） |
| Skeleton | 加载骨架 | 是（T002） |
| Empty | 空态展示 | 是（T002） |

### 7.2 业务组件（business/）
| 组件 | 用途 | 是否已存在 |
|------|------|-----------|
| InvoiceCard | 账单卡片 | 否 |

### 7.3 页面块组件（blocks/）
| 组件 | 用途 | 是否已存在 |
|------|------|-----------|
| InvoiceList | 账单列表 | 否 |
| InvoiceDetail | 账单详情 | 否 |
| InvoiceApplyForm | 发票申请表单 | 否 |

### 7.4 页面模板（layout/）
| 模板 | 用途 |
|------|------|
| ListPageShell | 列表页骨架 |

### 7.5 需新建组件
| 组件 | 所属分层 | 说明 |
|------|----------|------|
| InvoiceCard | business | 账单卡片 |
| InvoiceList | blocks | 账单列表 |
| InvoiceDetail | blocks | 账单详情 |
| InvoiceApplyForm | blocks | 发票申请表单 |

## 8. API 接口 ⭐

### 8.1 新增接口

#### 账单列表查询
- **Method**: GET
- **Path**: `/api/v1/billing/invoices`
- **请求类型**: `{ page?: number; pageSize?: number; status?: string }`
- **响应类型**: `{ items: InvoiceItem[]; total: number }`
- **权限**: 用户（已登录）

**响应示例**:
```json
{
  "success": true,
  "data": {
    "items": [
      {
        "id": "inv-001",
        "invoiceId": "INV-202604-001",
        "orderId": "order-001",
        "planId": "pro",
        "planName": "专业版",
        "amount": 99,
        "currency": "CNY",
        "status": "paid",
        "paymentMethod": "alipay",
        "paidAt": "2026-04-24T10:30:00Z",
        "createdAt": "2026-04-24T10:00:00Z",
        "periodStart": "2026-04-24",
        "periodEnd": "2027-04-23"
      }
    ],
    "total": 5
  }
}
```

#### 发票列表查询
- **Method**: GET
- **Path**: `/api/v1/billing/invoices/records`
- **响应类型**: `{ items: Invoice[]; total: number }`
- **权限**: 用户（已登录）

#### 发票申请
- **Method**: POST
- **Path**: `/api/v1/billing/invoices/apply`
- **请求类型**: `InvoiceApplyRequest`
- **响应类型**: `InvoiceApplyResponse`
- **权限**: 用户（已登录）

**请求示例**:
```json
{
  "invoiceType": "enterprise",
  "title": "某某科技有限公司",
  "taxNumber": "91110000XXXXXXXX",
  "email": "finance@example.com",
  "address": "北京市朝阳区",
  "phone": "010-12345678",
  "bankName": "中国工商银行",
  "bankAccount": "6222XXXXXXXX",
  "orderIds": ["order-001"]
}
```

**响应示例**:
```json
{
  "success": true,
  "data": {
    "invoiceId": "inv-20260424-001",
    "status": "pending",
    "estimatedCompletion": "2026-04-25T10:00:00Z"
  }
}
```

#### 账单下载
- **Method**: GET
- **Path**: `/api/v1/billing/invoices/:id/download`
- **响应类型**: `Blob`
- **权限**: 用户（已登录）

### 8.2 修改接口
无

### 8.3 接口定义位置
- 文件：`packages/shared/src/api/services.ts`
- 命名：`userApi.billing.invoices`, `userApi.billing.invoiceRecords`, `userApi.billing.applyInvoice`, `userApi.billing.downloadInvoice`

## 9. 验收标准 ⭐

### 9.1 功能验收
- [ ] 账单列表展示正常
- [ ] 账单详情展示正常
- [ ] 发票申请正常
- [ ] 发票状态跟踪正常
- [ ] 账单下载正常（PDF）
- [ ] 发票下载正常（PDF）
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
- [ ] 下载响应 < 3s
- [ ] 无内存泄漏（useEffect 清理）

### 9.4 兼容性验收
- [ ] 桌面端 ≥1200px 正常显示
- [ ] 用户端移动端核心功能可用

## 10. 风险评估

### 10.1 是否影响现网 ⭐
- [x] **否** — 全新功能，不影响现有页面

### 10.2 是否依赖其他任务先完成 ⭐
- [x] **是** — 依赖任务：
  - [x] `T019` — 计费概览页 — 待开发

### 10.3 技术风险
| 风险 | 可能性 | 影响 | 应对措施 |
|------|--------|------|----------|
| 发票信息错误 | 中 | 中 | 信息校验 |
| 下载文件大 | 低 | 中 | 流式下载 |
| 发票重复申请 | 中 | 中 | 后端幂等校验 |

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

- 发票申请需 1-3 个工作日处理
- 支持个人和企业发票
- 账单和发票支持 PDF 下载

---

**任务状态**：待评审 → 评审通过 → 开发中 → 自测完成 → CR 中 → 联调中 → 已上线
