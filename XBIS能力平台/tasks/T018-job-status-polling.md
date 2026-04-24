# 任务状态轮询

## 1. 基本信息

| 字段 | 内容 |
|------|------|
| 功能名称 ⭐ | 任务状态轮询 |
| 任务编号 | T018 |
| 所属模块 ⭐ | M5 任务系统 |
| 优先级 | P0 |
| 指派给 | @frontend-dev |
| 预计工期 | 2 人日 |
| 创建日期 | 2026-04-23 |
| 目标上线日期 | 2026-05-12 |

## 2. 需求背景

### 2.1 问题描述
当前项目缺乏任务状态自动刷新机制，用户需要：
- 实时了解任务状态变化
- 自动刷新任务列表
- 自动刷新任务详情

### 2.2 目标用户
- 终端用户：监控任务状态
- 开发者：了解任务执行进度

### 2.3 预期效果
- 提供自动状态刷新机制
- 支持列表页和详情页
- 智能轮询策略

## 3. 页面位置 ⭐

### 3.1 用户端
| 页面 | 路由 | 入口 |
|------|------|------|
| 任务列表 | `/jobs` | 自动刷新 |
| 任务详情 | `/jobs/:id` | 自动刷新 |

### 3.2 管理端（如适用）
无

### 3.3 页面原型/设计稿
- 参考：任务列表页和任务详情页

## 4. 功能范围

### 4.1 包含功能
- [ ] 任务列表自动刷新
- [ ] 任务详情自动刷新
- [ ] 智能轮询策略
- [ ] 状态变化通知
- [ ] 手动刷新
- [ ] 轮询开关

### 4.2 不包含功能（明确排除）
- WebSocket 实时推送（V2 迭代）
- 桌面通知（V2 迭代）
- 邮件通知（V2 迭代）

## 5. 数据结构 ⭐

### 5.1 新增/修改的数据模型

```typescript
// 轮询配置
export interface PollingConfig {
  enabled: boolean;
  interval: number;
  maxInterval: number;
  backoffMultiplier: number;
  maxRetries: number;
}

// 轮询状态
export interface PollingState {
  isPolling: boolean;
  lastPollTime: string;
  nextPollTime: string;
  pollCount: number;
  errorCount: number;
}

// 状态变化事件
export interface StatusChangeEvent {
  jobId: string;
  previousStatus: JobStatus;
  currentStatus: JobStatus;
  changedAt: string;
}
```

### 5.2 数据库变更（如适用）
无

### 5.3 类型定义位置
- 文件：`packages/shared/src/types/index.ts`
- 命名：`PollingConfig`, `PollingState`, `StatusChangeEvent`

## 6. 交互流程

### 6.1 主流程
```
[页面加载] ──► [启动轮询] ──► [定时查询状态] ──► [状态变化?] ──► [更新UI] ──► [继续轮询]
```

### 6.2 异常流程
| 场景 | 触发条件 | 系统行为 | 用户反馈 |
|------|----------|----------|----------|
| 轮询失败 | 网络错误 | 增加间隔，继续轮询 | 静默处理 |
| 连续失败 | 失败次数超过阈值 | 停止轮询 | 提示手动刷新 |
| 任务完成 | 状态为终态 | 停止轮询 | 状态标识 |
| 页面隐藏 | 切换标签页 | 暂停轮询 | 无感知 |

### 6.3 边界情况
- 大量任务：仅轮询可见任务
- 轮询频率：智能调整
- 页面隐藏：暂停轮询

## 7. 依赖组件 ⭐

### 7.1 基础组件（base/）
| 组件 | 用途 | 是否已存在 |
|------|------|-----------|
| Badge | 状态标识 | 是（T002） |
| Toast | 通知 | 是（T002） |

### 7.2 业务组件（business/）
| 组件 | 用途 | 是否已存在 |
|------|------|-----------|
| StatusBadge | 状态标识 | 是（T003） |

### 7.3 页面块组件（blocks/）
无

### 7.4 页面模板（layout/）
无

### 7.5 需新建组件
| 组件 | 所属分层 | 说明 |
|------|----------|------|
| PollingIndicator | blocks | 轮询状态指示器 |

## 8. API 接口 ⭐

### 8.1 新增接口

#### 任务状态查询（轻量）
- **Method**: GET
- **Path**: `/api/v1/jobs/:id/status`
- **响应类型**: `{ status: JobStatus; updatedAt: string }`
- **权限**: 用户（已登录）

**响应示例**:
```json
{
  "success": true,
  "data": {
    "status": "running",
    "updatedAt": "2026-04-24T10:00:05Z"
  }
}
```

#### 批量任务状态查询
- **Method**: POST
- **Path**: `/api/v1/jobs/status`
- **请求类型**: `{ jobIds: string[] }`
- **响应类型**: `{ jobId: string; status: JobStatus; updatedAt: string }[]`
- **权限**: 用户（已登录）

**请求示例**:
```json
{
  "jobIds": ["job-001", "job-002"]
}
```

**响应示例**:
```json
{
  "success": true,
  "data": [
    { "jobId": "job-001", "status": "completed", "updatedAt": "2026-04-24T10:00:05Z" },
    { "jobId": "job-002", "status": "running", "updatedAt": "2026-04-24T10:00:03Z" }
  ]
}
```

### 8.2 修改接口
无

### 8.3 接口定义位置
- 文件：`packages/shared/src/api/services.ts`
- 命名：`userApi.jobs.status`, `userApi.jobs.batchStatus`

## 9. 验收标准 ⭐

### 9.1 功能验收
- [ ] 任务列表自动刷新正常
- [ ] 任务详情自动刷新正常
- [ ] 智能轮询策略正常（初始 3s，逐步增加）
- [ ] 状态变化 UI 更新正常
- [ ] 手动刷新正常
- [ ] 轮询开关正常
- [ ] 页面隐藏暂停轮询
- [ ] 页面显示恢复轮询

### 9.2 技术验收
- [ ] TypeScript 类型完整，无 `any`
- [ ] 使用 Design Tokens，无散落样式
- [ ] 组件复用符合分层规范
- [ ] API 错误处理完整

### 9.3 性能验收
- [ ] 轮询间隔 ≥ 3s
- [ ] 轮询请求 < 100ms
- [ ] 无内存泄漏（useEffect 清理）
- [ ] 页面隐藏时无轮询请求

### 9.4 兼容性验收
- [ ] 桌面端 ≥1200px 正常显示
- [ ] 用户端移动端核心功能可用

## 10. 风险评估

### 10.1 是否影响现网 ⭐
- [x] **否** — 全新功能，不影响现有页面

### 10.2 是否依赖其他任务先完成 ⭐
- [x] **是** — 依赖任务：
  - [x] `T015` — 任务列表页 — 待开发

### 10.3 技术风险
| 风险 | 可能性 | 影响 | 应对措施 |
|------|--------|------|----------|
| 轮询频率过高 | 中 | 中 | 智能间隔，最小 3s |
| 内存泄漏 | 低 | 高 | useEffect 清理 |
| 并发请求 | 中 | 中 | 请求去重 |

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

- 轮询间隔策略：初始 3s，每次增加 2s，最大 30s
- 终态任务停止轮询：completed/failed/cancelled/timeout
- 页面隐藏使用 Page Visibility API 暂停轮询

---

**任务状态**：待评审 → 评审通过 → 开发中 → 自测完成 → CR 中 → 联调中 → 已上线
