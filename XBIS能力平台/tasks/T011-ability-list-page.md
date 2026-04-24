# 能力中心列表页

## 1. 基本信息

| 字段 | 内容 |
|------|------|
| 功能名称 ⭐ | 能力中心列表页 |
| 任务编号 | T011 |
| 所属模块 ⭐ | M4 能力平台 |
| 优先级 | P0 |
| 指派给 | @frontend-dev |
| 预计工期 | 3 人日 |
| 创建日期 | 2026-04-23 |
| 目标上线日期 | 2026-05-08 |

## 2. 需求背景

### 2.1 问题描述
当前项目的能力展示分散，缺乏统一的能力中心入口，需要：
- 统一的能力列表展示页面
- 支持分类筛选和标签过滤
- 支持关键词搜索

### 2.2 目标用户
- 终端用户：浏览和发现能力
- 开发者：查找需要的能力

### 2.3 预期效果
- 提供清晰的能力中心入口
- 支持多种筛选方式
- 支持能力卡片展示

## 3. 页面位置 ⭐

### 3.1 用户端
| 页面 | 路由 | 入口 |
|------|------|------|
| 能力中心 | `/abilities` | 主导航「能力中心」 |

### 3.2 管理端（如适用）
无

### 3.3 页面原型/设计稿
- 参考：`packages/pages/ability/AbilityListTemplate`

## 4. 功能范围

### 4.1 包含功能
- [ ] 能力网格展示
- [ ] 分类筛选（侧边栏）
- [ ] 标签过滤（顶部）
- [ ] 关键词搜索
- [ ] 排序（热门/最新/名称）
- [ ] 分页/无限滚动
- [ ] 空态/加载态/错误态

### 4.2 不包含功能（明确排除）
- 能力详情展示（由 T012 实现）
- 能力测试（由 T013 实现）
- 能力接入（由 T014 实现）

## 5. 数据结构 ⭐

### 5.1 新增/修改的数据模型

```typescript
// 能力卡片数据（列表项）
export interface AbilityCardItem {
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
  pricingType: 'free' | 'per_call' | 'package' | 'overage';
  unitPrice?: number;
  icon?: string;
  version: string;
  callCount: number;
  rating: number;
  createdAt: string;
}

// 筛选条件
export interface AbilityFilterState {
  category?: string;
  tags: string[];
  keyword: string;
  status: 'published' | 'draft' | 'deprecated' | 'all';
  sortBy: 'popular' | 'latest' | 'name';
  sortOrder: 'asc' | 'desc';
}

// 分类项
export interface CategoryItem {
  key: string;
  label: string;
  count: number;
  icon?: string;
}

// 标签项
export interface TagItem {
  key: string;
  label: string;
  count: number;
}
```

### 5.2 数据库变更（如适用）
无

### 5.3 类型定义位置
- 文件：`packages/shared/src/types/index.ts`
- 命名：`AbilityCardItem`, `AbilityFilterState`, `CategoryItem`, `TagItem`

## 6. 交互流程

### 6.1 主流程
```
[用户进入能力中心] ──► [加载分类和标签] ──► [加载能力列表] ──► [用户筛选/搜索] ──► [更新列表]
```

### 6.2 异常流程
| 场景 | 触发条件 | 系统行为 | 用户反馈 |
|------|----------|----------|----------|
| 加载失败 | 网络错误 | 显示错误态 | 提示刷新 |
| 无数据 | 筛选条件无匹配 | 显示空态 | 提示调整筛选 |
| 搜索无结果 | 关键词无匹配 | 显示空态 | 提示更换关键词 |

### 6.3 边界情况
- 大量分类：支持折叠和展开
- 大量标签：支持横向滚动
- 超长名称：截断显示 tooltip
- 暗色模式：自动适配

## 7. 依赖组件 ⭐

### 7.1 基础组件（base/）
| 组件 | 用途 | 是否已存在 |
|------|------|-----------|
| Button | 操作按钮 | 是（T002） |
| Input | 搜索框 | 是（T002） |
| Card | 能力卡片容器 | 是（T002） |
| Tag | 标签展示 | 是（T002） |
| Badge | 状态标识 | 是（T002） |
| Skeleton | 加载骨架 | 是（T002） |
| Empty | 空态展示 | 是（T002） |
| Pagination | 分页 | 是（T002） |

### 7.2 业务组件（business/）
| 组件 | 用途 | 是否已存在 |
|------|------|-----------|
| AbilityCard | 能力卡片 | 是（T003） |
| StatusBadge | 状态标识 | 是（T003） |

### 7.3 页面块组件（blocks/）
| 组件 | 用途 | 是否已存在 |
|------|------|-----------|
| AbilityGrid | 能力网格 | 否 |
| FilterSidebar | 筛选侧边栏 | 否 |
| TagFilterBar | 标签过滤栏 | 否 |
| SearchBar | 搜索栏 | 否 |

### 7.4 页面模板（layout/）
| 模板 | 用途 |
|------|------|
| ListPageShell | 列表页骨架 |

### 7.5 需新建组件
| 组件 | 所属分层 | 说明 |
|------|----------|------|
| AbilityGrid | blocks | 能力网格布局 |
| FilterSidebar | blocks | 筛选侧边栏 |
| TagFilterBar | blocks | 标签过滤栏 |
| SearchBar | blocks | 搜索栏 |

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

### 8.2 修改接口
无

### 8.3 接口定义位置
- 文件：`packages/shared/src/api/services.ts`
- 命名：`userApi.abilities.list`

## 9. 验收标准 ⭐

### 9.1 功能验收
- [ ] 能力网格展示正常，卡片信息完整
- [ ] 分类筛选正常，支持单选和多选
- [ ] 标签过滤正常，支持多选
- [ ] 关键词搜索正常，支持实时搜索
- [ ] 排序功能正常（热门/最新/名称）
- [ ] 分页功能正常
- [ ] 空态/加载态/错误态完整
- [ ] 点击卡片进入能力详情页

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
- [ ] 搜索响应 < 300ms
- [ ] 无内存泄漏（useEffect 清理）

### 9.4 兼容性验收
- [ ] 桌面端 ≥1200px 正常显示
- [ ] 用户端移动端核心功能可用

## 10. 风险评估

### 10.1 是否影响现网 ⭐
- [x] **否** — 全新功能，不影响现有页面

### 10.2 是否依赖其他任务先完成 ⭐
- [x] **是** — 依赖任务：
  - [x] `T003` — 业务组件库 — 待开发
  - [x] `T004` — 页面模板 — 待开发
  - [x] `T008` — ability API 层 — 待开发

### 10.3 技术风险
| 风险 | 可能性 | 影响 | 应对措施 |
|------|--------|------|----------|
| 大量数据渲染 | 中 | 中 | 虚拟滚动 + 分页 |
| 筛选条件复杂 | 低 | 中 | 状态管理清晰 |
| 移动端适配 | 中 | 低 | 响应式布局 |

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

- 参考实现：`packages/pages/ability/AbilityListTemplate`
- 能力卡片需展示调用次数和评分
- 支持暗色模式

---

**任务状态**：待评审 → 评审通过 → 开发中 → 自测完成 → CR 中 → 联调中 → 已上线
