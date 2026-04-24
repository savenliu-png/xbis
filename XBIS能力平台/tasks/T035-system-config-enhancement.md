# 系统配置增强

## 1. 基本信息

| 字段 | 内容 |
|------|------|
| 功能名称 ⭐ | 系统配置增强 |
| 任务编号 | T035 |
| 所属模块 ⭐ | M3 API 层 |
| 优先级 | P1 |
| 指派给 | @backend-dev |
| 预计工期 | 3 人日 |
| 创建日期 | 2026-04-23 |
| 目标上线日期 | 2026-05-20 |

## 2. 需求背景

### 2.1 问题描述
当前项目缺乏系统配置管理功能，需要：
- 变更日志
- 首页推荐配置
- 分类管理

### 2.2 目标用户
- 管理员：配置系统
- 运营人员：管理首页内容

### 2.3 预期效果
- 提供系统配置管理
- 支持首页推荐配置
- 支持分类管理

## 3. 页面位置 ⭐

### 3.1 用户端
无

### 3.2 管理端
| 页面 | 路由 | 入口 |
|------|------|------|
| 系统配置 | `/admin/settings` | 管理端导航「系统配置」 |

### 3.3 页面原型/设计稿
- 参考：`packages/pages/admin/SystemSettings`

## 4. 功能范围

### 4.1 包含功能
- [ ] 变更日志
- [ ] 首页推荐配置
- [ ] 分类管理
- [ ] 系统参数配置
- [ ] 配置版本管理

### 4.2 不包含功能（明确排除）
- 多环境配置（V2 迭代）
- 配置回滚（V2 迭代）
- 配置审批（V2 迭代）

## 5. 数据结构 ⭐

### 5.1 新增/修改的数据模型

```typescript
// 系统配置
export interface SystemConfig {
  id: string;
  key: string;
  value: any;
  description?: string;
  type: 'string' | 'number' | 'boolean' | 'json';
  category: string;
  updatedAt: string;
  updatedBy?: string;
}

// 变更日志
export interface ChangeLog {
  id: string;
  entityType: string;
  entityId: string;
  action: 'create' | 'update' | 'delete';
  changes: Record<string, { old: any; new: any }>;
  operatorId: string;
  operatorName: string;
  createdAt: string;
}

// 首页推荐
export interface HomeRecommendation {
  id: string;
  type: 'ability' | 'category' | 'banner';
  targetId: string;
  sortOrder: number;
  isActive: boolean;
  startAt?: string;
  endAt?: string;
}

// 分类管理
export interface CategoryManagement {
  id: string;
  key: string;
  label: string;
  description?: string;
  icon?: string;
  sortOrder: number;
  isActive: boolean;
  abilityCount: number;
}
```

### 5.2 数据库变更（如适用）
| 表名 | 操作 | 字段 | 类型 | 说明 |
|------|------|------|------|------|
| system_configs | CREATE | — | — | 系统配置表 |
| change_logs | CREATE | — | — | 变更日志表 |
| home_recommendations | CREATE | — | — | 首页推荐表 |
| categories | CREATE | — | — | 分类表 |

### 5.3 类型定义位置
- 文件：`packages/shared/src/types/index.ts`
- 命名：`SystemConfig`, `ChangeLog`, `HomeRecommendation`, `CategoryManagement`

## 6. 交互流程

### 6.1 主流程
```
[管理员进入系统配置] ──► [加载配置] ──► [展示数据] ──► [管理员编辑] ──► [保存配置]
```

### 6.2 异常流程
| 场景 | 触发条件 | 系统行为 | 用户反馈 |
|------|----------|----------|----------|
| 加载失败 | 网络错误 | 显示错误态 | 提示刷新 |
| 保存失败 | 数据错误 | 返回错误 | 提示修改 |
| 权限不足 | 非管理员 | 返回 403 | 提示无权限 |

### 6.3 边界情况
- 配置冲突：后端校验
- 大量配置：支持分页
- 配置缓存：定时刷新

## 7. 依赖组件 ⭐

无

## 8. API 接口 ⭐

### 8.1 新增接口

#### 系统配置查询
- **Method**: GET
- **Path**: `/admin-api/v1/settings`
- **响应类型**: `{ items: SystemConfig[]; total: number }`
- **权限**: 管理员

**响应示例**:
```json
{
  "success": true,
  "data": {
    "items": [
      {
        "id": "cfg-001",
        "key": "max_concurrent_jobs",
        "value": 100,
        "description": "最大并发任务数",
        "type": "number",
        "category": "system",
        "updatedAt": "2026-04-24T10:00:00Z",
        "updatedBy": "admin"
      }
    ],
    "total": 20
  }
}
```

#### 系统配置更新
- **Method**: PUT
- **Path**: `/admin-api/v1/settings/:id`
- **请求类型**: `{ value: any }`
- **响应类型**: `SystemConfig`
- **权限**: 管理员

#### 变更日志查询
- **Method**: GET
- **Path**: `/admin-api/v1/changelogs`
- **请求类型**: `{ entityType?: string; page?: number; pageSize?: number }`
- **响应类型**: `{ items: ChangeLog[]; total: number }`
- **权限**: 管理员

#### 首页推荐查询
- **Method**: GET
- **Path**: `/admin-api/v1/recommendations`
- **响应类型**: `{ items: HomeRecommendation[]; total: number }`
- **权限**: 管理员

#### 首页推荐更新
- **Method**: PUT
- **Path**: `/admin-api/v1/recommendations`
- **请求类型**: `HomeRecommendation[]`
- **响应类型**: `HomeRecommendation[]`
- **权限**: 管理员

#### 分类管理查询
- **Method**: GET
- **Path**: `/admin-api/v1/categories`
- **响应类型**: `{ items: CategoryManagement[]; total: number }`
- **权限**: 管理员

#### 分类管理更新
- **Method**: PUT
- **Path**: `/admin-api/v1/categories/:id`
- **请求类型**: `CategoryManagement`
- **响应类型**: `CategoryManagement`
- **权限**: 管理员

### 8.2 修改接口
无

### 8.3 接口定义位置
- 文件：`packages/shared/src/api/services.ts`
- 命名：`adminApi.settings.list`, `adminApi.settings.update`, `adminApi.changelogs`, `adminApi.recommendations`, `adminApi.categories`

## 9. 验收标准 ⭐

### 9.1 功能验收
- [ ] 系统配置查询正常
- [ ] 系统配置更新正常
- [ ] 变更日志记录正常
- [ ] 首页推荐配置正常
- [ ] 分类管理正常
- [ ] 配置版本管理正常

### 9.2 技术验收
- [ ] TypeScript 类型完整，无 `any`
- [ ] 配置校验完整
- [ ] 错误处理规范
- [ ] 接口文档（Swagger）生成

### 9.3 性能验收
- [ ] 配置查询 < 100ms
- [ ] 配置更新 < 500ms
- [ ] 支持配置缓存

### 9.4 兼容性验收
- [ ] 配置兼容旧版本
- [ ] 支持配置热更新

## 10. 风险评估

### 10.1 是否影响现网 ⭐
- [x] **否** — 全新功能，不影响现有页面

### 10.2 是否依赖其他任务先完成 ⭐
- [x] **否** — 可独立开发

### 10.3 技术风险
| 风险 | 可能性 | 影响 | 应对措施 |
|------|--------|------|----------|
| 配置错误 | 中 | 高 | 配置校验 |
| 缓存不一致 | 中 | 中 | 缓存刷新 |
| 并发修改 | 低 | 中 | 乐观锁 |

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

- 配置变更需记录日志
- 支持配置热更新
- 分类管理支持排序

---

**任务状态**：待评审 → 评审通过 → 开发中 → 自测完成 → CR 中 → 联调中 → 已上线
