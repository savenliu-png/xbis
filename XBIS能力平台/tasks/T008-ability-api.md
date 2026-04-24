# ability API 层

## 1. 基本信息

| 字段 | 内容 |
|------|------|
| 功能名称 ⭐ | ability API 层 |
| 任务编号 | T008 |
| 所属模块 ⭐ | M3 API 层 |
| 优先级 | P0 |
| 指派给 | @backend-dev |
| 预计工期 | 3 人日 |
| 创建日期 | 2026-04-23 |
| 目标上线日期 | 2026-05-05 |

## 2. 需求背景

### 2.1 问题描述
当前项目使用 api_market 相关接口，但随着架构向能力平台迁移，需要：
- 新的 ability 相关接口
- 兼容旧接口，逐步迁移

### 2.2 目标用户
- 前端开发者：调用 API 获取能力数据
- 后端开发者：维护 API 接口

### 2.3 预期效果
- 提供完整的 ability CRUD 接口
- 支持分类、标签、搜索
- 兼容旧 api_market 接口

## 3. 页面位置 ⭐

### 3.1 用户端
无

### 3.2 管理端
无

### 3.3 页面原型/设计稿
- 参考：`packages/shared/src/api/services.ts`

## 4. 功能范围

### 4.1 包含功能
- [ ] 能力列表查询（/api/v1/abilities）
- [ ] 能力详情查询（/api/v1/abilities/:id）
- [ ] 能力分类查询（/api/v1/abilities/categories）
- [ ] 能力标签查询（/api/v1/abilities/tags）
- [ ] 能力创建（/api/v1/abilities）
- [ ] 能力更新（/api/v1/abilities/:id）
- [ ] 能力删除（/api/v1/abilities/:id）

### 4.2 不包含功能（明确排除）
- 能力发布流程（由管理端实现）
- 能力审核流程（由管理端实现）

## 5. 数据结构 ⭐

### 5.1 新增/修改的数据模型

```typescript
// 能力列表查询参数
export interface AbilityListParams {
  category?: string;
  tags?: string[];
  keyword?: string;
  status?: 'published' | 'draft' | 'deprecated';
  page?: number;
  pageSize?: number;
}

// 能力列表响应
export interface AbilityListResponse {
  items: Ability[];
  total: number;
  categories: { key: string; label: string; count: number }[];
  tags: { key: string; label: string }[];
}

// 能力详情响应
export interface AbilityDetailResponse {
  ability: Ability;
  versions: AbilityVersion[];
  uiSchema?: AbilityUiSchema;
}

// 能力创建请求
export interface AbilityCreateRequest {
  name: string;
  displayName: string;
  description: string;
  category: string;
  tags: string[];
  riskLevel: 'low' | 'medium' | 'high' | 'critical';
  executionMode: 'sync' | 'async' | 'both' | 'review-only';
  pricingType: 'free' | 'per_call' | 'package' | 'overage';
  unitPrice?: number;
  requestSchema?: Record<string, any>;
  responseSchema?: Record<string, any>;
}

// 能力更新请求
export interface AbilityUpdateRequest {
  displayName?: string;
  description?: string;
  category?: string;
  tags?: string[];
  riskLevel?: 'low' | 'medium' | 'high' | 'critical';
  executionMode?: 'sync' | 'async' | 'both' | 'review-only';
  status?: 'published' | 'draft' | 'deprecated';
  pricingType?: 'free' | 'per_call' | 'package' | 'overage';
  unitPrice?: number;
  requestSchema?: Record<string, any>;
  responseSchema?: Record<string, any>;
}
```

### 5.2 数据库变更（如适用）
无

### 5.3 类型定义位置
- 文件：`packages/shared/src/types/index.ts`
- 命名：`AbilityListParams`, `AbilityListResponse`, `AbilityDetailResponse`, `AbilityCreateRequest`, `AbilityUpdateRequest`

## 6. 交互流程

### 6.1 主流程
```
[前端请求 API] ──► [后端校验参数] ──► [查询数据库] ──► [返回数据]
```

### 6.2 异常流程
| 场景 | 触发条件 | 系统行为 | 用户反馈 |
|------|----------|----------|----------|
| 参数错误 | 缺少必填参数 | 返回 400 | 提示参数错误 |
| 能力不存在 | 查询不存在的 abilityId | 返回 404 | 提示能力不存在 |
| 权限不足 | 无权限访问 | 返回 403 | 提示权限不足 |

### 6.3 边界情况
- 大数据量：支持分页和缓存
- 高并发：支持限流和降级

## 7. 依赖组件 ⭐

无

## 8. API 接口 ⭐

### 8.1 新增接口

#### 能力列表查询
- **Method**: GET
- **Path**: `/api/v1/abilities`
- **请求类型**: `AbilityListParams`
- **响应类型**: `AbilityListResponse`
- **权限**: 用户（已登录）

**请求示例**:
```json
{
  "category": "ai",
  "tags": ["sync", "free"],
  "keyword": "文本",
  "page": 1,
  "pageSize": 20
}
```

**响应示例**:
```json
{
  "success": true,
  "data": {
    "items": [...],
    "total": 128,
    "categories": [
      { "key": "all", "label": "全部", "count": 128 },
      { "key": "ai", "label": "AI 模型", "count": 24 }
    ],
    "tags": [
      { "key": "sync", "label": "同步" },
      { "key": "async", "label": "异步" }
    ]
  }
}
```

#### 能力详情查询
- **Method**: GET
- **Path**: `/api/v1/abilities/:id`
- **响应类型**: `AbilityDetailResponse`
- **权限**: 用户（已登录）

#### 能力创建
- **Method**: POST
- **Path**: `/api/v1/abilities`
- **请求类型**: `AbilityCreateRequest`
- **响应类型**: `Ability`
- **权限**: 管理员

#### 能力更新
- **Method**: PUT
- **Path**: `/api/v1/abilities/:id`
- **请求类型**: `AbilityUpdateRequest`
- **响应类型**: `Ability`
- **权限**: 管理员

#### 能力删除
- **Method**: DELETE
- **Path**: `/api/v1/abilities/:id`
- **权限**: 管理员

### 8.2 修改接口
| 接口 | 变更内容 | 兼容性 |
|------|----------|--------|
| `/portal-api/v1/api-market/*` | 路径迁移为 `/api/v1/abilities/*` | 保留旧端点，内部转发 |

### 8.3 接口定义位置
- 文件：`packages/shared/src/api/services.ts`
- 命名：`userApi.abilities.list`, `userApi.abilities.get`, `userApi.abilities.create`, `userApi.abilities.update`, `userApi.abilities.delete`

## 9. 验收标准 ⭐

### 9.1 功能验收
- [ ] 能力列表查询支持分类/标签/关键词筛选
- [ ] 能力详情查询返回完整信息
- [ ] 能力创建/更新/删除正常
- [ ] 旧接口兼容，内部转发到新接口
- [ ] 接口响应时间 < 300ms

### 9.2 技术验收
- [ ] TypeScript 类型完整，无 `any`
- [ ] 接口参数校验完整
- [ ] 错误处理规范
- [ ] 接口文档（Swagger）生成

### 9.3 性能验收
- [ ] 列表查询 < 300ms
- [ ] 详情查询 < 200ms
- [ ] 支持缓存

### 9.4 兼容性验收
- [ ] 旧接口可正常访问
- [ ] 新旧接口数据一致

## 10. 风险评估

### 10.1 是否影响现网 ⭐
- [x] **否** — 新接口，不影响现有页面

### 10.2 是否依赖其他任务先完成 ⭐
- [x] **是** — 依赖任务：
  - [x] `T005` — ability 数据模型 — 待开发

### 10.3 技术风险
| 风险 | 可能性 | 影响 | 应对措施 |
|------|--------|------|----------|
| 旧接口兼容问题 | 中 | 高 | 保留旧端点，内部转发 |
| 性能问题 | 中 | 中 | 添加缓存和索引 |

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

- 旧接口保留 1 个版本后移除
- 新接口使用 RESTful 规范
- 建议添加接口版本控制

---

**任务状态**：待评审 → 评审通过 → 开发中 → 自测完成 → CR 中 → 联调中 → 已上线
