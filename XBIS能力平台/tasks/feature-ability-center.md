# 能力中心（Ability Center）

## 1. 基本信息

| 字段 | 内容 |
|------|------|
| 功能名称 ⭐ | 能力中心 |
| 任务编号 | FEATURE-202604-001 |
| 所属模块 ⭐ | 用户端 — 能力平台 |
| 优先级 | P0 |
| 指派给 | @frontend-dev |
| 预计工期 | 5 人日 |
| 创建日期 | 2026-04-23 |
| 目标上线日期 | 2026-05-15 |

## 2. 需求背景

### 2.1 问题描述
当前项目使用 "API 市场"（ApiMarket）概念，随着架构向 "能力平台" 迁移，需要将接口/能力以新的形态展示给用户。用户需要一个统一入口来浏览、搜索、测试和接入平台提供的各类能力。

### 2.2 目标用户
- 终端用户（开发者）：浏览能力、在线测试、接入使用
- 终端用户（企业）：查看能力详情、评估风险、批量接入

### 2.3 预期效果
- 用户可以在统一入口浏览所有可用能力
- 支持分类筛选、关键词搜索、标签过滤
- 支持在线测试能力效果
- 支持一键接入能力到自有应用

## 3. 页面位置 ⭐

### 3.1 用户端
| 页面 | 路由 | 入口 |
|------|------|------|
| 能力列表页 | `/abilities` | 顶部导航「能力中心」 |
| 能力详情页 | `/abilities/:abilityId` | 能力列表页点击卡片 |
| 能力测试页 | `/abilities/:abilityId/test` | 能力详情页「在线测试」按钮 |
| 能力接入页 | `/abilities/:abilityId/subscribe` | 能力详情页「立即接入」按钮 |

### 3.2 管理端（如适用）
| 页面 | 路由 | 入口 |
|------|------|------|
| 能力管理 | `/admin/abilities` | 侧边栏「能力管理」 |
| 能力审核 | `/admin/abilities/reviews` | 侧边栏「内容与审核」 |

### 3.3 页面原型/设计稿
- Figma: `https://figma.com/xbis/ability-center`
- 参考实现: `packages/pages/ability/AbilityListTemplate`

## 4. 功能范围

### 4.1 包含功能
- [ ] 能力网格展示（卡片形式，支持分类筛选）
- [ ] 能力搜索（关键词搜索能力名称/描述）
- [ ] 分类侧边栏（全部/数据处理/AI模型/自然语言等）
- [ ] 标签过滤（同步/异步/免费/实时等）
- [ ] 能力详情展示（名称、描述、风险等级、执行模式、定价）
- [ ] Schema 预览（请求参数、响应参数）
- [ ] 在线测试（填写参数、实时调用、查看结果）
- [ ] 能力接入流程（选择套餐、确认接入、获取密钥）
- [ ] 详情预览面板（列表页点击卡片右侧预览）

### 4.2 不包含功能（明确排除）
- 能力发布流程（管理端功能，不在本任务范围）
- 能力版本管理（V2 迭代）
- 能力评价/评论系统（V2 迭代）

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
  items: ApiMarketItem[];
  total: number;
  categories: { key: string; label: string; count: number }[];
  tags: { key: string; label: string }[];
}

// 能力详情响应（复用现有 ApiMarketItem）
// 字段已存在于 packages/shared/src/types/index.ts

// 能力测试请求
export interface AbilityTestRequest {
  abilityId: string;
  parameters: Record<string, any>;
}

// 能力测试响应
export interface AbilityTestResponse {
  success: boolean;
  result?: Record<string, any>;
  error?: string;
  executionTime: number;
}

// 能力接入请求
export interface SubscribeAbilityRequest {
  abilityId: string;
  planId?: string;
  callbackUrl?: string;
}

// 能力接入响应
export interface SubscribeAbilityResponse {
  success: boolean;
  subscriptionId: string;
  apiKey: string;
  quota: { total: number; used: number };
}
```

### 5.2 数据库变更（如适用）
| 表名 | 操作 | 字段 | 类型 | 说明 |
|------|------|------|------|------|
| abilities | ADD | `doc_markdown` | TEXT | 能力文档说明 |
| abilities | ADD | `request_schema` | JSONB | 请求参数 Schema |
| abilities | ADD | `response_schema` | JSONB | 响应参数 Schema |
| subscriptions | NEW | — | — | 用户能力订阅关系表 |

### 5.3 类型定义位置
- 文件：`packages/shared/src/types/index.ts`
- 命名：`AbilityListParams`, `AbilityListResponse`, `AbilityTestRequest`, `AbilityTestResponse`

## 6. 交互流程

### 6.1 主流程：浏览并接入能力
```
[用户访问能力中心] ──► [展示能力网格]
   │
   ├──► [点击分类筛选] ──► [更新能力列表]
   │
   ├──► [输入关键词搜索] ──► [实时搜索能力]
   │
   ├──► [点击标签过滤] ──► [更新能力列表]
   │
   └──► [点击能力卡片] ──► [右侧预览详情]
            │
            ├──► [点击「立即接入」] ──► [进入接入流程]
            │
            └──► [点击「在线测试」] ──► [打开测试面板]
                        │
                        └──► [填写参数] ──► [提交测试] ──► [展示结果]
```

### 6.2 异常流程
| 场景 | 触发条件 | 系统行为 | 用户反馈 |
|------|----------|----------|----------|
| 搜索无结果 | 关键词无匹配 | 展示空态 | "未找到相关能力，请尝试其他关键词" |
| 能力不存在 | 访问已删除能力ID | 404 页面 | "该能力不存在或已下架" |
| 测试失败 | 参数错误或接口异常 | 展示错误信息 | Alert 提示具体错误原因 |
| 接入失败 | 配额不足或权限不足 | 阻止接入 | Modal 提示升级套餐或联系管理员 |

### 6.3 边界情况
- 分类下无能力：展示空态，提供「查看全部」快捷入口
- 能力无 Schema：隐藏「在线测试」按钮
- 免费能力无定价信息：展示「免费」标签，隐藏价格
- 网络异常：保留已有数据，顶部 Alert 提示重试

## 7. 依赖组件 ⭐

### 7.1 基础组件（base/）
| 组件 | 用途 | 是否已存在 |
|------|------|-----------|
| Card | 能力卡片容器 | 是 |
| Tag | 状态/分类标签 | 是 |
| Button | 操作按钮 | 是 |
| Search | 搜索框 | 是 |
| Empty | 空状态 | 是 |
| Spinner | 加载状态 | 是 |
| Alert | 错误提示 | 是 |
| Tabs | 详情页标签切换 | 是 |

### 7.2 业务组件（business/）
| 组件 | 用途 | 是否已存在 |
|------|------|-----------|
| AbilityCard | 能力卡片展示 | 是 |
| RiskBadge | 风险等级标识 | 是 |
| ExecutionModeTag | 执行模式标签 | 是 |
| PriceDisplay | 价格展示 | 是 |
| AiSparkle | AI 能力标识 | 是 |

### 7.3 页面块组件（blocks/）
| 组件 | 用途 | 是否已存在 |
|------|------|-----------|
| AbilityGrid | 能力网格布局 | 是 |
| AbilityDetailPanel | 详情面板 | 是 |
| SchemaFormBuilder | 测试表单 | 是 |

### 7.4 页面模板（layout/）
| 模板 | 用途 |
|------|------|
| AbilityPageShell | 能力列表页骨架 |
| DetailPageShell | 能力详情页骨架 |
| FormPageShell | 能力接入表单页 |

### 7.5 需新建组件
| 组件 | 所属分层 | 说明 |
|------|----------|------|
| SubscribeFlowModal | blocks | 能力接入流程弹窗（选择套餐→确认→成功） |
| TestResultPanel | blocks | 测试结果展示面板 |

## 8. API 接口 ⭐

### 8.1 新增接口

#### 能力列表查询
- **Method**: GET
- **Path**: `/portal-api/v1/abilities`
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
- **Path**: `/portal-api/v1/abilities/:abilityId`
- **响应类型**: `ApiMarketItem`
- **权限**: 用户（已登录）

#### 能力在线测试
- **Method**: POST
- **Path**: `/portal-api/v1/abilities/:abilityId/test`
- **请求类型**: `AbilityTestRequest`
- **响应类型**: `AbilityTestResponse`
- **权限**: 用户（已登录）

#### 能力接入
- **Method**: POST
- **Path**: `/portal-api/v1/abilities/:abilityId/subscribe`
- **请求类型**: `SubscribeAbilityRequest`
- **响应类型**: `SubscribeAbilityResponse`
- **权限**: 用户（已登录）

### 8.2 修改接口
| 接口 | 变更内容 | 兼容性 |
|------|----------|--------|
| `/portal-api/v1/apis/*` | 路径迁移为 `/portal-api/v1/abilities/*` | 需同步更新前端调用 |

### 8.3 接口定义位置
- 文件：`packages/shared/src/api/services.ts`
- 命名：`userApi.abilities.list`, `userApi.abilities.get`, `userApi.abilities.test`, `userApi.abilities.subscribe`

## 9. 验收标准 ⭐

### 9.1 功能验收
- [ ] 能力网格正确展示所有已发布能力
- [ ] 分类筛选实时更新能力列表（无需刷新页面）
- [ ] 关键词搜索支持模糊匹配能力名称和描述
- [ ] 标签过滤支持多选，结果取交集
- [ ] 点击能力卡片右侧展示详情预览（sticky 定位）
- [ ] 详情页展示完整的能力信息、Schema、文档
- [ ] 在线测试支持填写参数并实时调用
- [ ] 测试结果正确展示响应数据和执行时间
- [ ] 接入流程完整：选择套餐 → 确认 → 获取密钥

### 9.2 技术验收
- [ ] TypeScript 类型完整，无 `any`
- [ ] 使用 Design Tokens，无散落样式
- [ ] 使用 `AbilityPageShell` 和 `DetailPageShell` 页面模板
- [ ] 页面状态机完整（loading/empty/error/idle）
- [ ] 组件复用符合分层规范（base → business → blocks）
- [ ] API 错误处理完整，用户可感知

### 9.3 性能验收
- [ ] 能力列表首屏加载 < 1.5s
- [ ] 分类筛选响应 < 300ms
- [ ] 搜索防抖 300ms，避免频繁请求

### 9.4 兼容性验收
- [ ] 桌面端 ≥1200px 三栏布局正常（分类 + 网格 + 预览）
- [ ] 用户端移动端分类折叠为顶部下拉，网格单列

## 10. 风险评估

### 10.1 是否影响现网 ⭐
- [ ] **否** — 全新功能，不影响现有页面
- [x] **是** — 影响范围：
  - 影响页面：原 `/api-market` 页面将被 `/abilities` 替换
  - 影响用户：全部用户（路由变更）
  - 回滚方案：保留原路由 302 重定向到新路由，持续 1 个版本后移除

### 10.2 是否依赖其他任务先完成 ⭐
- [ ] **否** — 可独立开发
- [x] **是** — 依赖任务：
  - [x] `FEATURE-202604-000` — Design Tokens 设计 — 已完成
  - [x] `FEATURE-202604-002` — 组件库搭建 — 已完成
  - [ ] `FEATURE-202604-003` — 后端 abilities 表迁移 — 开发中

### 10.3 技术风险
| 风险 | 可能性 | 影响 | 应对措施 |
|------|--------|------|----------|
| 后端 abilities 接口延迟交付 | 中 | 高 | 使用 Mock 数据并行开发前端 |
| Schema 表单动态渲染复杂 | 中 | 中 | 优先支持 string/number/boolean 类型 |
| 在线测试接口性能问题 | 低 | 高 | 增加超时控制和降级提示 |

## 11. 开发检查清单

### 11.1 开发前
- [x] 需求已评审并确认
- [x] 设计稿已确认
- [x] 数据结构和 API 已评审
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

- 本任务为旧 API 平台向能力平台迁移的 P0 任务
- 参考实现已存在于 `packages/pages/ability/AbilityListTemplate` 和 `AbilityDetailTemplate`
- 开发时优先复用已有模板代码

---

**任务状态**：待评审 → 评审通过 → 开发中 → 自测完成 → CR 中 → 联调中 → 已上线
