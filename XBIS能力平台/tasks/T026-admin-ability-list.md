# 管理端能力管理列表

## 1. 基本信息

| 字段 | 内容 |
|------|------|
| 功能名称 ⭐ | 管理端能力管理列表 |
| 任务编号 | T026 |
| 所属模块 ⭐ | M10 能力管理 |
| 优先级 | P1 |
| 指派给 | @frontend-dev |
| 预计工期 | 3 人日 |
| 创建日期 | 2026-04-23 |
| 目标上线日期 | 2026-05-18 |

## 2. 需求背景

### 2.1 问题描述
当前项目缺乏管理端能力管理页面，管理员需要：
- 查看能力列表
- 管理能力状态
- 查看调用量

### 2.2 目标用户
- 管理员：管理能力
- 运营人员：监控能力使用

### 2.3 预期效果
- 提供清晰的能力管理列表
- 支持状态管理
- 支持调用量统计

## 3. 页面位置 ⭐

### 3.1 用户端
无

### 3.2 管理端
| 页面 | 路由 | 入口 |
|------|------|------|
| 能力管理 | `/admin/abilities` | 管理端导航「能力管理」 |

### 3.3 页面原型/设计稿
- 参考：`packages/pages/admin/AbilityManagement`

## 4. 功能范围

### 4.1 包含功能
- [ ] 能力列表展示
- [ ] 名称/分类/状态/版本筛选
- [ ] 调用量统计
- [ ] 状态切换（启用/禁用）
- [ ] 编辑入口
- [ ] 删除能力
- [ ] 分页
- [ ] 空态/加载态/错误态

### 4.2 不包含功能（明确排除）
- 能力编辑（由 T027 实现）
- 能力创建（由 T027 实现）
- 能力审核（V2 迭代）

## 5. 数据结构 ⭐

### 5.1 新增/修改的数据模型

```typescript
// 管理能力项
export interface AdminAbilityItem {
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
  createdAt: string;
  updatedAt: string;
}

// 管理筛选条件
export interface AdminAbilityFilter {
  keyword?: string;
  category?: string;
  status?: 'published' | 'draft' | 'deprecated';
  isEnabled?: boolean;
  page?: number;
  pageSize?: number;
}

// 状态切换请求
export interface AbilityStatusToggleRequest {
  abilityId: string;
  isEnabled: boolean;
}
```

### 5.2 数据库变更（如适用）
无

### 5.3 类型定义位置
- 文件：`packages/shared/src/types/index.ts`
- 命名：`AdminAbilityItem`, `AdminAbilityFilter`, `AbilityStatusToggleRequest`

## 6. 交互流程

### 6.1 主流程
```
[管理员进入能力管理] ──► [加载能力列表] ──► [展示数据] ──► [管理员操作]
```

### 6.2 异常流程
| 场景 | 触发条件 | 系统行为 | 用户反馈 |
|------|----------|----------|----------|
| 加载失败 | 网络错误 | 显示错误态 | 提示刷新 |
| 无数据 | 无能力 | 显示空态 | 提示创建 |
| 切换失败 | 能力正在使用 | 返回错误 | 提示原因 |
| 删除失败 | 能力有关联任务 | 返回错误 | 提示先下架 |

### 6.3 边界情况
- 大量能力：支持分页
- 状态变化：实时更新
- 暗色模式：自动适配

## 7. 依赖组件 ⭐

### 7.1 基础组件（base/）
| 组件 | 用途 | 是否已存在 |
|------|------|-----------|
| Button | 操作按钮 | 是（T002） |
| Input | 搜索框 | 是（T002） |
| Table | 能力表格 | 是（T002） |
| Tag | 状态标签 | 是（T002） |
| Badge | 状态标识 | 是（T002） |
| Switch | 状态切换 | 是（T002） |
| Modal | 确认弹窗 | 是（T002） |
| Skeleton | 加载骨架 | 是（T002） |
| Empty | 空态展示 | 是（T002） |
| Pagination | 分页 | 是（T002） |

### 7.2 业务组件（business/）
| 组件 | 用途 | 是否已存在 |
|------|------|-----------|
| StatusBadge | 状态标识 | 是（T003） |

### 7.3 页面块组件（blocks/）
| 组件 | 用途 | 是否已存在 |
|------|------|-----------|
| AbilityTable | 能力表格 | 否 |
| FilterBar | 筛选栏 | 否 |

### 7.4 页面模板（layout/）
| 模板 | 用途 |
|------|------|
| AdminPageShell | 管理页骨架 |

### 7.5 需新建组件
| 组件 | 所属分层 | 说明 |
|------|----------|------|
| AbilityTable | blocks | 能力表格 |
| FilterBar | blocks | 筛选栏 |

## 8. API 接口 ⭐

### 8.1 新增接口

#### 管理能力列表查询
- **Method**: GET
- **Path**: `/admin-api/v1/abilities`
- **请求类型**: `AdminAbilityFilter`
- **响应类型**: `{ items: AdminAbilityItem[]; total: number }`
- **权限**: 管理员

**响应示例**:
```json
{
  "success": true,
  "data": {
    "items": [
      {
        "id": "ability-001",
        "abilityId": "text-generation",
        "name": "text-generation",
        "displayName": "文本生成",
        "category": "ai",
        "status": "published",
        "isEnabled": true,
        "version": "v1.2.0",
        "callCount": 12580,
        "successRate": 98.5,
        "averageLatency": 1200,
        "createdAt": "2026-01-15T00:00:00Z",
        "updatedAt": "2026-04-20T00:00:00Z"
      }
    ],
    "total": 128
  }
}
```

#### 能力状态切换
- **Method**: PUT
- **Path**: `/admin-api/v1/abilities/:id/status`
- **请求类型**: `AbilityStatusToggleRequest`
- **响应类型**: `AdminAbilityItem`
- **权限**: 管理员

#### 能力删除
- **Method**: DELETE
- **Path**: `/admin-api/v1/abilities/:id`
- **权限**: 管理员

### 8.2 修改接口
无

### 8.3 接口定义位置
- 文件：`packages/shared/src/api/services.ts`
- 命名：`adminApi.abilities.list`, `adminApi.abilities.toggleStatus`, `adminApi.abilities.delete`

## 9. 验收标准 ⭐

### 9.1 功能验收
- [ ] 能力列表展示正常
- [ ] 名称/分类/状态筛选正常
- [ ] 调用量统计展示正常
- [ ] 状态切换正常
- [ ] 编辑入口可点击
- [ ] 删除能力正常
- [ ] 分页功能正常
- [ ] 空态/加载态/错误态完整

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
- [ ] 无内存泄漏（useEffect 清理）

### 9.4 兼容性验收
- [ ] 桌面端 ≥1200px 正常显示
- [ ] 管理端无需移动端设计

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
| 误删除 | 中 | 高 | 二次确认 |
| 状态冲突 | 低 | 中 | 后端校验 |
| 数据量大 | 中 | 中 | 分页加载 |

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

- 删除能力需二次确认
- 状态切换实时生效
- 支持批量操作（V2 迭代）

---

**任务状态**：待评审 → 评审通过 → 开发中 → 自测完成 → CR 中 → 联调中 → 已上线
