# 管理端用户管理

## 1. 基本信息

| 字段 | 内容 |
|------|------|
| 功能名称 ⭐ | 管理端用户管理 |
| 任务编号 | T030 |
| 所属模块 ⭐ | M13 用户与订单 |
| 优先级 | P1 |
| 指派给 | @frontend-dev |
| 预计工期 | 3 人日 |
| 创建日期 | 2026-04-23 |
| 目标上线日期 | 2026-05-20 |

## 2. 需求背景

### 2.1 问题描述
当前项目缺乏管理端用户管理页面，管理员需要：
- 查看用户列表
- 查看用户详情
- 管理用户限制
- 查看用户订单

### 2.2 目标用户
- 管理员：管理用户
- 运营人员：查看用户数据

### 2.3 预期效果
- 提供清晰的用户管理
- 支持用户详情查看
- 支持订单管理

## 3. 页面位置 ⭐

### 3.1 用户端
无

### 3.2 管理端
| 页面 | 路由 | 入口 |
|------|------|------|
| 用户管理 | `/admin/users` | 管理端导航「用户管理」 |
| 用户详情 | `/admin/users/:id` | 用户列表点击 |

### 3.3 页面原型/设计稿
- 参考：`packages/pages/admin/UserManagement`

## 4. 功能范围

### 4.1 包含功能
- [ ] 用户列表展示
- [ ] 用户详情展示
- [ ] 用户状态管理（启用/禁用）
- [ ] 用户限制配置
- [ ] 用户订单列表
- [ ] 订单详情
- [ ] 空态/加载态/错误态

### 4.2 不包含功能（明确排除）
- 用户创建（V2 迭代）
- 用户导入（V2 迭代）
- 用户导出（V2 迭代）

## 5. 数据结构 ⭐

### 5.1 新增/修改的数据模型

```typescript
// 管理端用户项
export interface AdminUserItem {
  id: string;
  username: string;
  email: string;
  phone?: string;
  displayName: string;
  company?: string;
  status: 'active' | 'inactive' | 'banned';
  role: 'user' | 'admin';
  totalSpent: number;
  totalJobs: number;
  createdAt: string;
  lastLoginAt?: string;
}

// 用户详情
export interface UserDetailAdmin {
  user: AdminUserItem;
  orders: UserOrder[];
  jobs: RecentJob[];
  settings: UserProfile;
}

// 用户订单
export interface UserOrder {
  id: string;
  orderId: string;
  planId: string;
  planName: string;
  amount: number;
  currency: string;
  status: 'pending' | 'paid' | 'failed' | 'refunded';
  paidAt?: string;
  createdAt: string;
}

// 用户限制配置
export interface UserLimitConfig {
  maxConcurrency: number;
  maxDailyJobs: number;
  maxMonthlySpend: number;
  allowedAbilities: string[];
  blockedAbilities: string[];
}
```

### 5.2 数据库变更（如适用）
无

### 5.3 类型定义位置
- 文件：`packages/shared/src/types/index.ts`
- 命名：`AdminUserItem`, `UserDetailAdmin`, `UserOrder`, `UserLimitConfig`

## 6. 交互流程

### 6.1 主流程
```
[管理员进入用户管理] ──► [加载用户列表] ──► [展示数据] ──► [管理员操作]
```

### 6.2 异常流程
| 场景 | 触发条件 | 系统行为 | 用户反馈 |
|------|----------|----------|----------|
| 加载失败 | 网络错误 | 显示错误态 | 提示刷新 |
| 无用户 | 无注册用户 | 显示空态 | 提示正常 |
| 禁用失败 | 用户有进行中的任务 | 返回错误 | 提示先处理任务 |
| 配置失败 | 配置无效 | 返回错误 | 提示修改 |

### 6.3 边界情况
- 大量用户：支持分页
- 敏感信息：脱敏展示
- 暗色模式：自动适配

## 7. 依赖组件 ⭐

### 7.1 基础组件（base/）
| 组件 | 用途 | 是否已存在 |
|------|------|-----------|
| Button | 操作按钮 | 是（T002） |
| Input | 搜索框 | 是（T002） |
| Table | 用户表格 | 是（T002） |
| Tag | 状态标签 | 是（T002） |
| Badge | 状态标识 | 是（T002） |
| Switch | 状态切换 | 是（T002） |
| Modal | 确认弹窗 | 是（T002） |
| Tabs | 内容切换 | 是（T002） |
| Skeleton | 加载骨架 | 是（T002） |
| Empty | 空态展示 | 是（T002） |
| Pagination | 分页 | 是（T002） |

### 7.2 业务组件（business/）
| 组件 | 用途 | 是否已存在 |
|------|------|-----------|
| UserCard | 用户卡片 | 否 |

### 7.3 页面块组件（blocks/）
| 组件 | 用途 | 是否已存在 |
|------|------|-----------|
| UserTable | 用户表格 | 否 |
| UserDetailPanel | 用户详情面板 | 否 |
| OrderList | 订单列表 | 否 |
| LimitConfig | 限制配置 | 否 |

### 7.4 页面模板（layout/）
| 模板 | 用途 |
|------|------|
| AdminPageShell | 管理页骨架 |

### 7.5 需新建组件
| 组件 | 所属分层 | 说明 |
|------|----------|------|
| UserCard | business | 用户卡片 |
| UserTable | blocks | 用户表格 |
| UserDetailPanel | blocks | 用户详情面板 |
| OrderList | blocks | 订单列表 |
| LimitConfig | blocks | 限制配置 |

## 8. API 接口 ⭐

### 8.1 新增接口

#### 用户列表查询（管理端）
- **Method**: GET
- **Path**: `/admin-api/v1/users`
- **请求类型**: `{ page?: number; pageSize?: number; keyword?: string; status?: string }`
- **响应类型**: `{ items: AdminUserItem[]; total: number }`
- **权限**: 管理员

**响应示例**:
```json
{
  "success": true,
  "data": {
    "items": [
      {
        "id": "user-001",
        "username": "developer",
        "email": "dev@example.com",
        "phone": "13800138000",
        "displayName": "开发者",
        "company": "某某科技",
        "status": "active",
        "role": "user",
        "totalSpent": 999,
        "totalJobs": 12580,
        "createdAt": "2026-01-01T00:00:00Z",
        "lastLoginAt": "2026-04-24T10:00:00Z"
      }
    ],
    "total": 1280
  }
}
```

#### 用户详情查询（管理端）
- **Method**: GET
- **Path**: `/admin-api/v1/users/:id`
- **响应类型**: `UserDetailAdmin`
- **权限**: 管理员

#### 用户状态更新
- **Method**: PUT
- **Path**: `/admin-api/v1/users/:id/status`
- **请求类型**: `{ status: 'active' | 'inactive' | 'banned' }`
- **响应类型**: `AdminUserItem`
- **权限**: 管理员

#### 用户限制配置
- **Method**: PUT
- **Path**: `/admin-api/v1/users/:id/limits`
- **请求类型**: `UserLimitConfig`
- **响应类型**: `UserLimitConfig`
- **权限**: 管理员

### 8.2 修改接口
无

### 8.3 接口定义位置
- 文件：`packages/shared/src/api/services.ts`
- 命名：`adminApi.users.list`, `adminApi.users.get`, `adminApi.users.updateStatus`, `adminApi.users.updateLimits`

## 9. 验收标准 ⭐

### 9.1 功能验收
- [ ] 用户列表展示正常
- [ ] 用户详情展示正常
- [ ] 用户状态管理正常
- [ ] 用户限制配置正常
- [ ] 用户订单列表正常
- [ ] 订单详情正常
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
- [ ] 列表页支持虚拟滚动（>100 条数据时）
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
| 敏感信息泄露 | 低 | 高 | 数据脱敏 |
| 误操作 | 中 | 高 | 二次确认 |
| 数据量大 | 中 | 中 | 分页加载 |

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

- 敏感信息需脱敏展示
- 用户禁用需二次确认
- 支持暗色模式

---

**任务状态**：待评审 → 评审通过 → 开发中 → 自测完成 → CR 中 → 联调中 → 已上线
