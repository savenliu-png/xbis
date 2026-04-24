# ability 数据模型

## 1. 基本信息

| 字段 | 内容 |
|------|------|
| 功能名称 ⭐ | ability 数据模型 |
| 任务编号 | T005 |
| 所属模块 ⭐ | M2 数据层 |
| 优先级 | P0 |
| 指派给 | @backend-dev |
| 预计工期 | 3 人日 |
| 创建日期 | 2026-04-23 |
| 目标上线日期 | 2026-04-28 |

## 2. 需求背景

### 2.1 问题描述
当前项目使用 api_market_item 表存储能力信息，但随着架构向能力平台迁移，需要：
- 更丰富的能力定义字段
- 版本管理支持
- UI 配置支持

### 2.2 目标用户
- 后端开发者：使用统一的数据模型开发 API
- 前端开发者：依赖类型定义开发页面

### 2.3 预期效果
- 建立 ability 表，扩展 api_market_item 字段
- 支持能力版本管理
- 支持 UI 配置存储

## 3. 页面位置 ⭐

### 3.1 用户端
无

### 3.2 管理端
无

### 3.3 页面原型/设计稿
- 参考：`docs/方案&配置/01 xbis-front 系统重构方案.md`

## 4. 功能范围

### 4.1 包含功能
- [ ] 创建 ability 表
- [ ] 创建 ability_version 表
- [ ] 创建 ability_ui_schema 表
- [ ] 数据迁移脚本（api_market_item → ability）
- [ ] 双写机制

### 4.2 不包含功能（明确排除）
- 能力发布流程（由管理端实现）
- 能力审核流程（由管理端实现）

## 5. 数据结构 ⭐

### 5.1 新增/修改的数据模型

```typescript
// ability 表
export interface Ability {
  id: string;
  abilityId: string;
  name: string;
  displayName: string;
  description: string;
  category: string;
  tags: string[];
  riskLevel: 'low' | 'medium' | 'high' | 'critical';
  executionMode: 'sync' | 'async' | 'both' | 'review-only';
  status: 'published' | 'draft' | 'deprecated';
  isEnabled: boolean;
  pricingType: 'free' | 'per_call' | 'package' | 'overage';
  unitPrice?: number;
  freeQuota?: number;
  overagePrice?: number;
  requestSchema?: Record<string, any>;
  responseSchema?: Record<string, any>;
  docMarkdown?: string;
  icon?: string;
  version: string;
  createdAt: string;
  updatedAt: string;
}

// ability_version 表
export interface AbilityVersion {
  id: string;
  abilityId: string;
  version: string;
  changeLog: string;
  status: 'draft' | 'testing' | 'published' | 'deprecated';
  createdAt: string;
  publishedAt?: string;
}

// ability_ui_schema 表
export interface AbilityUiSchema {
  id: string;
  abilityId: string;
  version: string;
  uiSchema: Record<string, any>;
  formConfig?: Record<string, any>;
  createdAt: string;
  updatedAt: string;
}
```

### 5.2 数据库变更（如适用）
| 表名 | 操作 | 字段 | 类型 | 说明 |
|------|------|------|------|------|
| abilities | CREATE | — | — | 能力主表 |
| ability_versions | CREATE | — | — | 能力版本表 |
| ability_ui_schemas | CREATE | — | — | 能力 UI 配置表 |
| api_market_item | ADD | `ability_id` | VARCHAR(64) | 关联 ability |

### 5.3 类型定义位置
- 文件：`packages/shared/src/types/index.ts`
- 命名：`Ability`, `AbilityVersion`, `AbilityUiSchema`

## 6. 交互流程

### 6.1 主流程
```
[创建 ability 表] ──► [创建 ability_version 表] ──► [创建 ability_ui_schema 表] ──► [数据迁移]
```

### 6.2 异常流程
| 场景 | 触发条件 | 系统行为 | 用户反馈 |
|------|----------|----------|----------|
| 迁移失败 | 数据不一致 | 回滚，保留旧表 | 日志记录 |
| 双写失败 | 新表写入失败 | 记录日志，继续写入旧表 | 监控告警 |

### 6.3 边界情况
- 历史数据缺失字段：使用默认值
- 重复 abilityId：唯一索引校验

## 7. 依赖组件 ⭐

无

## 8. API 接口 ⭐

无

## 9. 验收标准 ⭐

### 9.1 功能验收
- [ ] ability 表创建成功，字段完整
- [ ] ability_version 表创建成功，支持版本管理
- [ ] ability_ui_schema 表创建成功，支持 UI 配置
- [ ] 数据迁移脚本可正常运行
- [ ] 双写机制验证通过
- [ ] 新旧表数据一致性校验通过

### 9.2 技术验收
- [ ] 数据库迁移可回滚
- [ ] 索引优化，查询性能 < 100ms
- [ ] 字段命名符合规范

### 9.3 性能验收
- [ ] 单表数据量 10w 时查询 < 100ms
- [ ] 迁移脚本执行时间 < 10 分钟

### 9.4 兼容性验收
- [ ] 旧代码可正常读取 api_market_item

## 10. 风险评估

### 10.1 是否影响现网 ⭐
- [x] **否（新表）** — 新建表，不影响现有数据

### 10.2 是否依赖其他任务先完成 ⭐
- [x] **否** — 可独立开发

### 10.3 技术风险
| 风险 | 可能性 | 影响 | 应对措施 |
|------|--------|------|----------|
| 数据迁移不一致 | 低 | 高 | 双写验证，每日比对 |
| 字段设计遗漏 | 中 | 中 | 预留扩展字段 |

## 11. 开发检查清单

### 11.1 开发前
- [x] 需求已评审并确认
- [x] 设计稿已确认
- [x] 数据结构和 API 已评审
- [x] 依赖任务已完成

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

- 参考现有 api_market_item 表结构
- 预留扩展字段，支持未来功能
- 双写期间监控数据一致性

---

**任务状态**：待评审 → 评审通过 → 开发中 → 自测完成 → CR 中 → 联调中 → 已上线
