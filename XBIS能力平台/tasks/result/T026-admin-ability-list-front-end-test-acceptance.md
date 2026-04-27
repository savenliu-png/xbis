# T026 管理端能力管理列表 - 前后端联调测试与验收报告

## 一、联调范围

| 类型 | 对象 | 是否新增 | 风险等级 | 是否必须联调 |
| ---- | ---- | -------- | -------- | ------------ |
| 页面入口 | /admin/abilities | 是 | 高 | ✅ |
| API | GET /admin-api/v1/abilities | 是 | 高 | ✅ |
| API | PUT /admin-api/v1/abilities/:id/status | 是 | 高 | ✅ |
| API | DELETE /admin-api/v1/abilities/:id | 是 | 中 | ✅ |
| API | POST /admin-api/v1/abilities/batch-delete | 是 | 高 | ✅ |
| API | POST /admin-api/v1/abilities/batch-status | 是 | 高 | ✅ |
| API | GET /admin-api/v1/categories | 否 | 低 | ✅ |
| 请求参数 | AbilityListParams（含 isEnabled） | 是 | 中 | ✅ |
| 响应结构 | AdminAbilityListResponse | 是 | 高 | ✅ |
| 类型定义 | AdminAbilityItem | 是 | 中 | ✅ |
| 类型定义 | AbilityStatusToggleRequest | 是 | 低 | ✅ |
| 类型定义 | BatchDeleteRequest | 是 | 低 | ✅ |
| 类型定义 | BatchStatusRequest | 是 | 低 | ✅ |
| 状态流 | published/draft/deprecated | 否 | 中 | ✅ |
| 状态流 | isEnabled true/false | 是 | 中 | ✅ |
| 权限 | adminAuthMiddleware | 否 | 低 | ✅ |
| 降级逻辑 | 关联统计字段可选 | 是 | 低 | ✅ |
| 旧接口兼容 | /admin-api/v1/apis（旧接口权限） | 否 | 中 | ✅ |

---

## 二、API契约核对

### 关键发现：后端未实现 `/abilities` 路由

经核实 `packages/server/src/routes/admin.ts` 和 `adminP0.ts`，**后端不存在任何 `/abilities` 路由处理器**。现有后端仅实现 `/apis` 路由（旧版接口权限管理），使用 `platform.apis` 数据库表。

| API | Method | 前端调用 | 后端契约 | 请求参数一致 | 响应字段一致 | 状态 |
| ---| ------ | -------- | -------- | ------------ | ------------ | ---- |
| GET /admin-api/v1/abilities | GET | ✅ adminApi.abilities.list() | ❌ **后端未实现** | N/A | N/A | ❌ 404 |
| PUT /admin-api/v1/abilities/:id/status | PUT | ✅ adminApi.abilities.toggleStatus() | ❌ **后端未实现** | N/A | N/A | ❌ 404 |
| DELETE /admin-api/v1/abilities/:id | DELETE | ✅ adminApi.abilities.delete() | ❌ **后端未实现** | N/A | N/A | ❌ 404 |
| POST /admin-api/v1/abilities/batch-delete | POST | ✅ adminApi.abilities.batchDelete() | ❌ **后端未实现** | N/A | N/A | ❌ 404 |
| POST /admin-api/v1/abilities/batch-status | POST | ✅ adminApi.abilities.batchStatus() | ❌ **后端未实现** | N/A | N/A | ❌ 404 |
| GET /admin-api/v1/categories | GET | ✅ adminApi.categories.list() | ✅ 后端已实现 | ✅ | ✅ | ✅ |

### 前端 API 定义与后端 `/apis` 路由的对比

| 对比项 | 前端 `/abilities` | 后端 `/apis` | 差异 |
|--------|-------------------|-------------|------|
| URL 前缀 | /admin-api/v1/abilities | /admin-api/v1/apis | 不同路径 |
| 列表查询参数 | keyword, category, status, isEnabled, page, pageSize | keyword, category, publishStatus, sourceType, page, pageSize | status vs publishStatus |
| 状态切换 | PUT /abilities/:id/status { isEnabled } | POST /apis/:apiId/status { status } | Method + 字段不同 |
| 批量操作 | batch-delete, batch-status | 无 | 前端新增 |
| 删除 | DELETE /abilities/:id | DELETE /apis/:apiId | 路径不同 |

---

## 三、数据与类型核对

### 3.1 AdminAbilityItem vs 后端 `platform.apis` 表字段

| 字段 | 前端 AdminAbilityItem | 后端 platform.apis | 问题 | 修复建议 |
| ---- | -------------------- | ------------------ | ---- | -------- |
| id | string | api_id (uuid) | 字段名不同 | 后端需映射 |
| abilityId | string | 无 | 前端有后端无 | 后端需新增 |
| name | string | api_name | 字段名不同 | 后端需映射 |
| displayName | string | display_name | 字段名不同 | 后端需映射 |
| category | string | category | ✅ 一致 | - |
| status | AbilityStatus | publish_status/status | 枚举值不同 | 后端需映射 |
| isEnabled | boolean | 无 | 前端有后端无 | 后端需新增 |
| version | string | 无 | 前端有后端无 | 后端需新增 |
| callCount | number | 无 | 前端有后端无 | 后端需 JOIN 统计 |
| successRate | number | 无 | 前端有后端无 | 后端需 JOIN 统计 |
| averageLatency | number | 无 | 前端有后端无 | 后端需 JOIN 统计 |
| subscriberCount | number? | 无 | 前端有后端无 | 后端需 JOIN 统计 |
| activeJobCount | number? | 无 | 前端有后端无 | 后端需 JOIN 统计 |
| totalJobCount | number? | 无 | 前端有后端无 | 后端需 JOIN 统计 |
| createdAt | string | created_at | ✅ 一致（snake→camel） | - |
| updatedAt | string | updated_at | ✅ 一致（snake→camel） | - |

### 3.2 AdminAbilityListResponse 字段缺失

| 字段 | 前端定义 | 后端 /apis 返回 | 问题 | 修复建议 |
| ---- | -------- | --------------- | ---- | -------- |
| items | AdminAbilityItem[] | rows[] | ✅ | - |
| total | number | count | ✅ | - |
| page | number? | page | ✅ 已修复（添加可选） | - |
| pageSize | number? | pageSize | ✅ 已修复（添加可选） | - |

### 3.3 枚举值对比

| 枚举 | 前端定义 | 后端 /apis 值 | 问题 |
| ---- | -------- | ------------- | ---- |
| AbilityStatus | published/draft/deprecated | active/draft/inactive/pending_review | ❌ 不一致 |
| isEnabled | boolean | 无对应字段 | ❌ 后端缺失 |

---

## 四、主流程联调

| 流程 | 操作 | 预期结果 | 实际结果 | 状态 |
| ---- | ---- | -------- | -------- | ---- |
| 打开页面 | 导航到 /admin/abilities | 加载列表 | 后端 404 → error 状态 | ❌ 后端未实现 |
| 加载数据 | GET /abilities | 返回列表 | 404 Not Found | ❌ 后端未实现 |
| 搜索能力 | 输入关键词 | 筛选结果 | 无法测试 | ⚠️ 待后端 |
| 状态切换 | 点击 Switch | 乐观更新+API | 404 → 回滚+error | ❌ 后端未实现 |
| 删除能力 | 点击删除+确认 | 弹窗+API | 404 → error | ❌ 后端未实现 |
| 批量操作 | 勾选+批量操作 | API+刷新 | 404 → error | ❌ 后端未实现 |
| 分页 | 翻页 | 下一页数据 | 无法测试 | ⚠️ 待后端 |
| 分类加载 | GET /categories | 下拉选项 | ✅ 后端已实现 | ✅ |

---

## 五、异常与边界测试

| 场景 | 后端返回 | 前端表现 | 是否通过 |
| ---- | -------- | -------- | -------- |
| 接口 404（当前实际状态） | 404 Not Found | catch → setPageStatus('error') → Alert + 重试 | ✅ 前端处理正确 |
| 接口 401 | 401 Unauthorized | client.ts 自动刷新 token / 跳转登录 | ✅ |
| 接口 403 | 403 Forbidden | catch → setPageStatus('error') | ✅ |
| 接口 500 | 500 Internal Error | catch → setPageStatus('error') | ✅ |
| 网络超时 | timeout 30s | catch → setPageStatus('error') | ✅ |
| 空数据 | { items: [], total: 0 } | setPageStatus('empty') → Empty 组件 | ✅ |
| 字段缺失 | item 缺少 subscriberCount | 可选字段，不展示关联影响 | ✅ 优雅降级 |
| 枚举未知值 | status='unknown' | AbilityStatusTag fallback → warning | ✅ |
| 重复提交 | 快速点击 Switch | toggleLoadingId 防止重复 | ✅ |
| 数值边界 | successRate=0, callCount=0 | 正确展示 0 | ✅ |
| 时间格式异常 | updatedAt=null | new Date(null) → 'Invalid Date' | ⚠️ 需修复 |

---

## 六、Bug列表

| Bug编号 | 严重级别 | 问题描述 | 复现步骤 | 责任归属 | 影响范围 | 修复建议 |
| ------- | -------- | -------- | -------- | -------- | -------- | -------- |
| FE-001 | **Blocker** | 后端未实现 /abilities 路由，所有 API 调用返回 404 | 打开 /admin/abilities 页面 | 后端 | 全部功能不可用 | 后端需实现 5 个 API 端点 |
| FE-002 | Medium | AdminAbilityListResponse 缺少 page/pageSize 字段 | 后端返回含 page 字段时前端未映射 | 前端 | 分页状态可能不同步 | 已修复：添加可选字段 |
| FE-003 | Low | updatedAt=null 时显示 'Invalid Date' | 后端返回 updatedAt=null | 前端 | 时间列显示异常 | 添加 null 检查 |
| FE-004 | Medium | AbilityStatus 枚举与后端 /apis 的状态值不一致 | 后端使用 active/inactive/pending_review | 契约不一致 | 状态筛选不匹配 | 需统一枚举定义 |

---

## 七、Bug修复内容

### 前端修复

#### FE-002: AdminAbilityListResponse 添加 page/pageSize

**问题原因**：`AbilityListResponse`（用户端）包含 `page`/`pageSize`，但 `AdminAbilityListResponse` 缺失。后端分页接口通常返回这两个字段。

**修改文件**：`packages/shared/src/types/ability.ts`

**修复代码**：
```typescript
export interface AdminAbilityListResponse {
  items: AdminAbilityItem[];
  total: number;
  page?: number;      // 新增
  pageSize?: number;  // 新增
}
```

#### FE-003: updatedAt null 检查

**问题原因**：`new Date(null)` 返回 `Invalid Date`，`toLocaleString` 输出 "Invalid Date"。

**修改文件**：`packages/components/blocks/AbilityTable/index.tsx`

**修复代码**：
```typescript
render: (value: unknown) => {
  if (!value) return '-';
  const d = new Date(String(value));
  return isNaN(d.getTime()) ? '-' : d.toLocaleString('zh-CN');
},
```

### 后端修复建议

#### FE-001: 后端需实现 5 个 /abilities API 端点

**需新增的路由**：

| Method | URL | 请求参数 | 响应结构 |
|--------|-----|---------|---------|
| GET | /admin-api/v1/abilities | AbilityListParams (query) | AdminAbilityListResponse |
| PUT | /admin-api/v1/abilities/:id/status | { isEnabled: boolean } (body) | { success: true } |
| DELETE | /admin-api/v1/abilities/:id | - | { success: true } |
| POST | /admin-api/v1/abilities/batch-delete | { ids: string[] } (body) | { success: true } |
| POST | /admin-api/v1/abilities/batch-status | { ids: string[], isEnabled: boolean } (body) | { success: true } |

**推荐 DTO**：

```typescript
// 列表查询参数
interface AbilityListQuery {
  keyword?: string;
  category?: string;
  status?: 'published' | 'draft' | 'deprecated';
  isEnabled?: boolean;
  page?: number;
  pageSize?: number;
}

// 列表响应
interface AbilityListResponse {
  items: AbilityListItem[];
  total: number;
  page: number;
  pageSize: number;
}

// 列表项
interface AbilityListItem {
  id: string;
  abilityId: string;
  name: string;
  displayName: string;
  category: string;
  status: 'published' | 'draft' | 'deprecated';
  isEnabled: boolean;
  version: string;
  callCount: number;
  successRate: number;
  averageLatency: number;
  subscriberCount?: number;
  activeJobCount?: number;
  totalJobCount?: number;
  createdAt: string;
  updatedAt: string;
}
```

**示例请求**：
```
GET /admin-api/v1/abilities?keyword=test&category=tools&status=published&isEnabled=true&page=1&pageSize=20
```

**示例响应**：
```json
{
  "success": true,
  "data": {
    "items": [
      {
        "id": "uuid-1",
        "abilityId": "ability-001",
        "name": "text_generator",
        "displayName": "文本生成器",
        "category": "tools",
        "status": "published",
        "isEnabled": true,
        "version": "1.2.0",
        "callCount": 15823,
        "successRate": 99.7,
        "averageLatency": 245,
        "subscriberCount": 42,
        "activeJobCount": 3,
        "totalJobCount": 1205,
        "createdAt": "2025-01-15T08:30:00Z",
        "updatedAt": "2025-04-20T14:22:00Z"
      }
    ],
    "total": 128,
    "page": 1,
    "pageSize": 20
  }
}
```

#### FE-004: 枚举统一建议

| 前端枚举 | 后端 /apis 枚举 | 建议统一值 |
|---------|----------------|-----------|
| published | active | **published**（更语义化） |
| draft | draft | draft |
| deprecated | inactive | **deprecated**（更明确） |
| - | pending_review | **pending_review**（新增） |

---

## 八、复测结果

| 测试项 | 结果 | 说明 |
| ------ | ---- | ---- |
| FE-002: AdminAbilityListResponse page/pageSize | ✅ | 已添加可选字段 |
| FE-003: updatedAt null 检查 | ✅ | 已添加 isNaN 检查 |
| FE-001: 后端 404 前端处理 | ✅ | catch → error 状态 + 重试按钮 |
| 主流程：页面打开 | ✅ | 页面可打开，显示 error 状态（后端 404） |
| 主流程：分类加载 | ✅ | categories API 正常 |
| 状态回归：loading/empty/error | ✅ | 三态完整 |
| 异常回归：404/500/timeout | ✅ | 均进入 error 状态 |
| 旧功能回归：/apis 页面 | ✅ | 不受影响 |
| 旧功能回归：variant="primary" | ✅ | 已在回归测试中修复 |
| Design Tokens | ✅ | 无硬编码值 |
| Business Services 层 | ✅ | 全部通过 adminApi |

---

## 九、功能验收结论

| 验收项 | 状态 | 说明 |
| ------ | ---- | ---- |
| 9.1 功能验收（8项） | ⚠️ 部分通过 | 前端逻辑完整，但后端未实现导致功能不可用 |
| 9.2 技术验收（6项） | ✅ 全部通过 | TypeScript/Design Tokens/模板/状态/组件复用/API规范 |
| 9.3 性能验收（3项） | ✅ 2/3 通过 | 虚拟滚动未实现（分页模式下不触发） |
| 9.4 兼容性验收（2项） | ✅ 全部通过 | |

👉 状态：**Accepted with notes**

### Notes

1. **后端 FE-001 为 Blocker**：后端必须实现 5 个 `/abilities` API 端点后，功能才可实际使用
2. **前端已完整实现**：所有前端逻辑、状态管理、错误处理、UI 组件均已完成，等待后端 API 上线即可工作
3. **前端 404 降级处理正确**：后端未实现时，页面显示 error 状态 + 重试按钮，不会白屏
4. **枚举统一 FE-004 需后端配合**：AbilityStatus 枚举需前后端统一

---

## 十、是否允许合并

| 决策项 | 结论 | 说明 |
| ------ | ---- | ---- |
| 是否允许合并 | **YES** | 前端代码完整，后端 404 有降级处理，不阻塞合并 |
| 是否允许进入下一任务 | **YES** | T027（能力编辑）可并行开发 |
| 是否需要继续修复 | **NO** | 前端无 Blocker/High Bug |
| 是否需要后端继续处理 | **YES** | 后端需实现 5 个 API 端点（FE-001） |
| 是否需要产品确认 | **YES** | AbilityStatus 枚举统一方案需产品确认（FE-004） |
