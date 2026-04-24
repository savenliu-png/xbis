# 数据迁移脚本

## 1. 基本信息

| 字段 | 内容 |
|------|------|
| 功能名称 ⭐ | 数据迁移脚本 |
| 任务编号 | T036 |
| 所属模块 ⭐ | M2 数据层 |
| 优先级 | P0 |
| 指派给 | @backend-dev |
| 预计工期 | 5 人日 |
| 创建日期 | 2026-04-23 |
| 目标上线日期 | 2026-05-05 |

## 2. 需求背景

### 2.1 问题描述
当前项目需要迁移历史数据：
- api_market_item → ability
- invocation → job
- 需要保证数据一致性
- 需要支持回滚

### 2.2 目标用户
- 后端开发者：执行迁移
- 运维人员：监控迁移

### 2.3 预期效果
- 提供完整的迁移脚本
- 支持数据校验
- 支持回滚

## 3. 页面位置 ⭐

### 3.1 用户端
无

### 3.2 管理端
无

### 3.3 页面原型/设计稿
- 参考：`scripts/migration/`

## 4. 功能范围

### 4.1 包含功能
- [ ] api_market_item → ability 迁移
- [ ] invocation → job 迁移
- [ ] 数据校验
- [ ] 数据修复
- [ ] 回滚脚本
- [ ] 迁移日志
- [ ] 双写机制

### 4.2 不包含功能（明确排除）
- 在线迁移（V2 迭代）
- 增量迁移（V2 迭代）
- 数据清洗（V2 迭代）

## 5. 数据结构 ⭐

### 5.1 新增/修改的数据模型

```typescript
// 迁移配置
export interface MigrationConfig {
  sourceTable: string;
  targetTable: string;
  batchSize: number;
  dryRun: boolean;
  skipValidation: boolean;
}

// 迁移结果
export interface MigrationResult {
  sourceTable: string;
  targetTable: string;
  totalRecords: number;
  migratedRecords: number;
  failedRecords: number;
  skippedRecords: number;
  startTime: string;
  endTime: string;
  duration: number;
  errors: MigrationError[];
}

// 迁移错误
export interface MigrationError {
  recordId: string;
  error: string;
  stack?: string;
}

// 数据校验结果
export interface ValidationResult {
  table: string;
  totalRecords: number;
  validRecords: number;
  invalidRecords: number;
  errors: ValidationError[];
}

// 校验错误
export interface ValidationError {
  recordId: string;
  field: string;
  expected: any;
  actual: any;
}
```

### 5.2 数据库变更（如适用）
| 表名 | 操作 | 字段 | 类型 | 说明 |
|------|------|------|------|------|
| migration_logs | CREATE | — | — | 迁移日志表 |

### 5.3 类型定义位置
- 文件：`packages/shared/src/types/index.ts`
- 命名：`MigrationConfig`, `MigrationResult`, `MigrationError`, `ValidationResult`, `ValidationError`

## 6. 交互流程

### 6.1 主流程
```
[开始迁移] ──► [读取源数据] ──► [转换数据] ──► [写入目标表] ──► [校验数据] ──► [记录日志]
```

### 6.2 异常流程
| 场景 | 触发条件 | 系统行为 | 用户反馈 |
|------|----------|----------|----------|
| 数据不一致 | 校验失败 | 记录错误 | 提示修复 |
| 写入失败 | 数据库错误 | 回滚批次 | 记录日志 |
| 内存不足 | 数据量大 | 分批处理 | 提示进度 |
| 迁移中断 | 进程崩溃 | 记录断点 | 支持续传 |

### 6.3 边界情况
- 大量数据：分批处理
- 数据冲突：跳过或覆盖
- 字段缺失：使用默认值

## 7. 依赖组件 ⭐

无

## 8. API 接口 ⭐

### 8.1 新增接口

#### 迁移状态查询
- **Method**: GET
- **Path**: `/admin-api/v1/migration/status`
- **响应类型**: `{ status: string; progress: number; result?: MigrationResult }`
- **权限**: 管理员

**响应示例**:
```json
{
  "success": true,
  "data": {
    "status": "completed",
    "progress": 100,
    "result": {
      "sourceTable": "api_market_item",
      "targetTable": "abilities",
      "totalRecords": 128,
      "migratedRecords": 128,
      "failedRecords": 0,
      "skippedRecords": 0,
      "startTime": "2026-04-24T10:00:00Z",
      "endTime": "2026-04-24T10:05:00Z",
      "duration": 300,
      "errors": []
    }
  }
}
```

#### 执行迁移
- **Method**: POST
- **Path**: `/admin-api/v1/migration/run`
- **请求类型**: `MigrationConfig`
- **响应类型**: `MigrationResult`
- **权限**: 管理员

#### 数据校验
- **Method**: POST
- **Path**: `/admin-api/v1/migration/validate`
- **请求类型**: `{ table: string }`
- **响应类型**: `ValidationResult`
- **权限**: 管理员

### 8.2 修改接口
无

### 8.3 接口定义位置
- 文件：`packages/shared/src/api/services.ts`
- 命名：`adminApi.migration.status`, `adminApi.migration.run`, `adminApi.migration.validate`

## 9. 验收标准 ⭐

### 9.1 功能验收
- [ ] api_market_item → ability 迁移正常
- [ ] invocation → job 迁移正常
- [ ] 数据校验正常
- [ ] 数据修复正常
- [ ] 回滚脚本正常
- [ ] 迁移日志记录正常
- [ ] 双写机制正常

### 9.2 技术验收
- [ ] TypeScript 类型完整，无 `any`
- [ ] 迁移脚本可重复执行
- [ ] 支持断点续传
- [ ] 错误处理规范

### 9.3 性能验收
- [ ] 迁移速度 > 1000 条/分钟
- [ ] 内存使用 < 1GB
- [ ] 不影响现网性能

### 9.4 兼容性验收
- [ ] 旧数据可正常读取
- [ ] 新数据格式正确
- [ ] 双写期间数据一致

## 10. 风险评估

### 10.1 是否影响现网 ⭐
- [x] **否** — 迁移脚本独立运行，不影响现有页面

### 10.2 是否依赖其他任务先完成 ⭐
- [x] **是** — 依赖任务：
  - [x] `T005` — ability 数据模型 — 待开发
  - [x] `T006` — job 数据模型 — 待开发

### 10.3 技术风险
| 风险 | 可能性 | 影响 | 应对措施 |
|------|--------|------|----------|
| 数据丢失 | 低 | 高 | 备份 + 校验 |
| 数据不一致 | 中 | 高 | 双写验证 |
| 迁移失败 | 中 | 高 | 回滚脚本 |
| 性能影响 | 中 | 中 | 低峰期执行 |

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

- 迁移前必须备份数据
- 建议在低峰期执行迁移
- 双写期间监控数据一致性
- 迁移后需验证数据完整性

---

**任务状态**：待评审 → 评审通过 → 开发中 → 自测完成 → CR 中 → 联调中 → 已上线
