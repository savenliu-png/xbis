# T036 数据迁移脚本 - 最终设计方案（Final）

## 1. 页面结构

### 1.1 层级结构
本任务主要为后端脚本任务，前端仅提供迁移状态监控页面。

```
AdminLayout
└── AdminPageShell
    ├── PageHeader
    │   ├── Title: "数据迁移"
    │   └── Breadcrumb: 概览 / 数据迁移
    └── PageShell
        ├── MigrationStatusCard
        │   ├── 迁移状态（未开始/进行中/已完成/失败）
        │   ├── 进度条
        │   └── 统计信息
        ├── MigrationControls
        │   ├── [开始迁移] 按钮
        │   ├── [校验数据] 按钮
        │   └── [回滚] 按钮
        └── MigrationLog
            └── 迁移日志表格
```

### 1.2 模板使用
| 区域 | 模板/组件 | 说明 |
|------|----------|------|
| 外层 | AdminPageShell | 管理端骨架 |
| 内容 | PageShell | 标准页面 |

---

## 2. 组件拆分

### 2.1 base/ 层组件（已存在）
| 组件 | 用途 |
|------|------|
| Button, Progress, Table, Tag, Badge, Skeleton, Empty | 标准用法 |

### 2.2 blocks/ 层组件（需新建）
| 组件 | 用途 | 说明 |
|------|------|------|
| MigrationStatusCard | 迁移状态卡片 | 状态 + 进度 + 统计 |
| MigrationControls | 迁移控制 | 操作按钮组 |
| MigrationLog | 迁移日志 | 日志表格 |

---

## 3. 数据流

### 3.1 状态查询
```
[进入 /admin/migration]
    │
    ▼
[API: GET /admin-api/v1/migration/status]
    │
    ▼
[渲染迁移状态]
```

### 3.2 执行迁移
```
[点击「开始迁移」]
    │
    ▼
[检查备份状态]
    │
    ▼
[确认 Modal]
    │
    ▼
[API: POST /admin-api/v1/migration/run]
    │
    ├── 成功 → 开始轮询状态
    └── 失败 → 显示错误
```

### 3.3 数据校验
```
[点击「校验数据」]
    │
    ▼
[API: POST /admin-api/v1/migration/validate]
    │
    ▼
[展示校验结果]
```

---

## 4. 状态管理

```typescript
interface MigrationPageState {
  pageState: 'loading' | 'idle' | 'running' | 'error';
  status: 'not_started' | 'running' | 'completed' | 'failed';
  progress: number;
  result: MigrationResult | null;
  logs: MigrationLogItem[];
  error: ApiError | null;
}
```

---

## 5. API 调用

| API | Method | Path | 说明 |
|-----|--------|------|------|
| 迁移状态 | GET | `/admin-api/v1/migration/status` | 查询迁移状态 |
| 执行迁移 | POST | `/admin-api/v1/migration/run` | 开始迁移 |
| 数据校验 | POST | `/admin-api/v1/migration/validate` | 校验数据 |

### 数据校验规则
```typescript
interface MigrationValidationRule {
  name: string;
  sourceQuery: string;
  targetQuery: string;
  comparator: 'count' | 'sum' | 'exact';
}

const validationRules: MigrationValidationRule[] = [
  { name: '记录数一致', sourceQuery: 'SELECT COUNT(*) FROM api_market_item', targetQuery: 'SELECT COUNT(*) FROM abilities', comparator: 'count' },
  { name: '字段值一致', sourceQuery: 'SELECT id, name FROM api_market_item', targetQuery: 'SELECT id, display_name FROM abilities', comparator: 'exact' }
];
```

### 回滚限制
```typescript
interface RollbackLimitation {
  maxRollbackTime: number; // 迁移后多长时间内可回滚（如 24 小时）
  allowNewData: boolean;   // 是否允许迁移后有新数据写入
  backupRequired: boolean; // 是否要求备份
}
```

---

## 6. 用户交互流程

```
[进入数据迁移页]
    │
    ▼
[查看当前迁移状态]
    │
    ├── 未开始
    │       │
    │       ▼
    │   [检查备份状态]
    │       │
    │       ▼
    │   [点击「开始迁移」]
    │       │
    │       ▼
    │   [确认 Modal]
    │       │
    │       ▼
    │   [执行迁移]
    │
    ├── 进行中
    │       │
    │       ▼
    │   [查看进度]
    │   [查看日志]
    │
    └── 已完成
            │
            ▼
        [查看结果]
        [点击「校验数据」]
        [点击「回滚」（限时）]
```

### 强制数据备份
迁移前必须：
1. 全量备份源数据库
2. 备份目标数据库（如已有数据）
3. 记录备份时间点和位置
4. 验证备份可恢复性

---

## 7. 边界与异常

| 场景 | 处理方式 |
|------|----------|
| 迁移失败 | 显示错误详情 + 回滚按钮（限时） |
| 校验失败 | 显示错误记录 + 修复建议 |
| 迁移中断 | 支持断点续传 |
| 无权限 | 403 页面 |
| 未备份 | 禁止开始迁移 |

---

## 8. 性能优化

- 迁移状态轮询（5 秒间隔）
- 日志分页加载
- 进度条平滑动画
- 迁移期间服务策略：
  - 方案A（停机迁移）：迁移期间服务不可用，显示维护页面
  - 方案B（在线迁移）：双写机制，迁移期间同时写入旧表和新表
  - 推荐方案A（简单可控）

---

## 9. 风险点

| 风险 | 说明 | 应对方案 |
|------|------|----------|
| 数据丢失 | 迁移过程可能丢失数据 | 备份；双写机制；校验 |
| 数据不一致 | 迁移后数据不一致 | 校验脚本；双写验证 |
| 迁移失败 | 中途失败需要回滚 | 回滚脚本；事务机制；限时回滚 |
| 性能影响 | 迁移影响现网性能 | 低峰期执行；限流；停机迁移 |

---

## 10. 开发步骤

### Step 1: 迁移脚本（3 天）
- [ ] api_market_item → ability 迁移
- [ ] invocation → job 迁移
- [ ] 数据校验逻辑
- [ ] 回滚脚本

### Step 2: 后端 API（1 天）
- [ ] 迁移状态查询
- [ ] 执行迁移接口
- [ ] 数据校验接口

### Step 3: 前端监控页（0.5 天）
- [ ] 迁移状态展示
- [ ] 进度条
- [ ] 日志查看

### Step 4: 测试（0.5 天）
- [ ] 端到端测试
- [ ] 回滚测试

---

## 附录：修改文件清单

### 新增文件
```
packages/pages/admin/Migration/
├── index.tsx
├── MigrationPage.tsx
└── components/
    ├── MigrationStatusCard.tsx
    ├── MigrationControls.tsx
    └── MigrationLog.tsx

scripts/migration/
├── migrate-abilities.ts
├── migrate-jobs.ts
├── validate-data.ts
└── rollback.ts
```

### 修改文件
```
packages/shared/src/types/index.ts
packages/shared/src/api/services.ts
packages/router/admin.tsx
```
