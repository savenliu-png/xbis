# callback 机制

## 1. 基本信息

| 字段 | 内容 |
|------|------|
| 功能名称 ⭐ | callback 机制 |
| 任务编号 | T034 |
| 所属模块 ⭐ | M3 API 层 |
| 优先级 | P1 |
| 指派给 | @backend-dev |
| 预计工期 | 3 人日 |
| 创建日期 | 2026-04-23 |
| 目标上线日期 | 2026-05-12 |

## 2. 需求背景

### 2.1 问题描述
当前项目缺乏任务完成后的回调机制，需要：
- 任务完成回调
- 回调重试
- 回调记录

### 2.2 目标用户
- 开发者：接收任务完成通知
- 后端开发者：维护回调机制

### 2.3 预期效果
- 提供可靠的回调机制
- 支持重试策略
- 支持回调记录查询

## 3. 页面位置 ⭐

### 3.1 用户端
无

### 3.2 管理端
无

### 3.3 页面原型/设计稿
- 参考：`packages/shared/src/api/services.ts`

## 4. 功能范围

### 4.1 包含功能
- [ ] 任务完成回调
- [ ] 回调重试（指数退避）
- [ ] 回调记录
- [ ] 回调状态查询
- [ ] 回调配置

### 4.2 不包含功能（明确排除）
- WebSocket 推送（V2 迭代）
- 事件总线（V2 迭代）
- 消息队列（V2 迭代）

## 5. 数据结构 ⭐

### 5.1 新增/修改的数据模型

```typescript
// 回调记录
export interface CallbackRecord {
  id: string;
  jobId: string;
  callbackUrl: string;
  payload: Record<string, unknown>;
  status: 'pending' | 'success' | 'failed' | 'retrying';
  attemptCount: number;
  maxAttempts: number;
  nextRetryAt?: string;
  lastError?: string;
  createdAt: string;
  completedAt?: string;
}

// 回调配置
export interface CallbackConfig {
  maxRetries: number;
  retryInterval: number;
  backoffMultiplier: number;
  timeout: number;
  enabled: boolean;
}

// 回调请求（后端内部使用）
export interface CallbackRequest {
  jobId: string;
  callbackUrl: string;
  payload: Record<string, unknown>;
}

// 回调响应
export interface CallbackResponse {
  recordId: string;
  status: 'pending' | 'success' | 'failed' | 'retrying';
  message: string;
}
```

### 5.2 数据库变更（如适用）
| 表名 | 操作 | 字段 | 类型 | 说明 |
|------|------|------|------|------|
| callback_records | CREATE | — | — | 回调记录表 |

### 5.3 类型定义位置
- 文件：`packages/shared/src/types/index.ts`
- 命名：`CallbackRecord`, `CallbackConfig`, `CallbackRequest`, `CallbackResponse`

## 6. 交互流程

### 6.1 主流程
```
[任务完成] ──► [触发回调] ──► [发送请求] ──► [成功?] ──► [记录状态]
```

### 6.2 异常流程
| 场景 | 触发条件 | 系统行为 | 用户反馈 |
|------|----------|----------|----------|
| 回调失败 | 网络错误 | 重试 | 记录日志 |
| 重试耗尽 | 超过最大重试次数 | 标记失败 | 记录日志 |
| 回调超时 | 响应超时 | 重试 | 记录日志 |
| 无效 URL | URL 格式错误 | 标记失败 | 记录日志 |

### 6.3 边界情况
- 大量回调：支持批量处理
- 回调风暴：支持限流
- 回调顺序：保证顺序

## 7. 依赖组件 ⭐

无

## 8. API 接口 ⭐

### 8.1 新增接口

#### 回调记录查询
- **Method**: GET
- **Path**: `/api/v1/callbacks`
- **请求类型**: `{ jobId?: string; status?: string; page?: number; pageSize?: number }`
- **响应类型**: `{ items: CallbackRecord[]; total: number; page: number; pageSize: number }`
- **权限**: 用户（已登录）

**响应示例**:
```json
{
  "success": true,
  "data": {
        "items": [
          {
            "id": "cb-001",
            "jobId": "job-001",
            "callbackUrl": "https://example.com/callback",
            "payload": { "jobId": "job-001", "status": "completed" },
            "status": "success",
            "attemptCount": 1,
            "maxAttempts": 3,
            "createdAt": "2026-04-24T10:00:00Z",
            "completedAt": "2026-04-24T10:00:01Z"
          }
        ],
        "total": 10,
        "page": 1,
        "pageSize": 20
      }
}
```

#### 手动触发回调
- **Method**: POST
- **Path**: `/api/v1/callbacks/:id/retry`
- **响应类型**: `CallbackResponse`
- **权限**: 用户（已登录）

### 8.2 修改接口
无

### 8.3 接口定义位置
- 文件：`packages/shared/src/api/services.ts`
- 命名：`userApi.callbacks.list`, `userApi.callbacks.retry`

## 9. 验收标准 ⭐

### 9.1 功能验收
- [ ] 任务完成回调正常
- [ ] 回调重试正常（指数退避）
- [ ] 回调记录正常
- [ ] 回调状态查询正常
- [ ] 手动重试正常
- [ ] 回调配置正常

### 9.2 技术验收
- [ ] TypeScript 类型完整，无 `any`
- [ ] 回调幂等性保证
- [ ] 错误处理规范
- [ ] 接口文档（Swagger）生成

### 9.3 性能验收
- [ ] 回调发送 < 1s
- [ ] 重试间隔正确
- [ ] 支持并发回调

### 9.4 兼容性验收
- [ ] 回调兼容 HTTP/HTTPS
- [ ] 支持 JSON 格式

## 10. 风险评估

### 10.1 是否影响现网 ⭐
- [x] **否** — 全新功能，不影响现有页面

### 10.2 是否依赖其他任务先完成 ⭐
- [x] **是** — 依赖任务：
  - [x] `T009` — job API 层 — 待开发

### 10.3 技术风险
| 风险 | 可能性 | 影响 | 应对措施 |
|------|--------|------|----------|
| 回调风暴 | 中 | 高 | 限流和队列 |
| 重试风暴 | 中 | 高 | 指数退避 |
| 回调安全 | 中 | 高 | 签名验证 |

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

- 回调重试策略：指数退避，最大 3 次
- 回调超时：30 秒
- 支持回调签名验证

---

**任务状态**：待评审 → 评审通过 → 开发中 → 自测完成 → CR 中 → 联调中 → 已上线
