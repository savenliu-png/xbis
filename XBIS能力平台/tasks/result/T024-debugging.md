# T024 用户端导航重构 — 接口联调检查报告（D1）

> 检查日期: 2026-04-26
> 检查角色: 资深后端架构师 + 前端联调负责人 + API契约审查官

---

## 1️⃣ API契约一致性

### T024 涉及的 API 调用

| API | URL | Method | 前端调用 | 后端路由 | 一致性 |
|-----|-----|--------|---------|---------|--------|
| 通知列表 | `/portal-api/v1/notifications` | GET | `userApi.notifications.list()` | `userP0.ts L568` | ✅ 一致 |

**说明**：T024 任务卡明确标注"无新增 API、无修改 API"。导航配置为静态配置（NAV_ITEMS），唯一涉及的 API 调用是获取未读通知数。

---

## 2️⃣ 返回结构匹配

### `GET /portal-api/v1/notifications` 返回结构

**后端实际返回**（userP0.ts L568-577 + notificationCenter.ts L19-35）：

```typescript
{
  success: true,
  data: {
    items: [
      {
        id: string,
        title: string,
        content: string,
        category: string,          // 'system' | 'security' | 'billing' | ...
        priority: string,          // 'low' | 'normal' | 'high'
        actionUrl: string | null,
        relatedResourceType: string | null,
        relatedResourceId: string | null,
        channel: string | null,
        senderAdminId: string | null,
        senderName: string | null,
        readAt: string | null,     // ✅ 驼峰命名
        sentAt: string | null,
        createdAt: string,         // ✅ 驼峰命名
        isRead: boolean,           // ✅ 布尔值
      }
    ],
    unreadCount: number            // ✅ 后端直接返回未读数
  }
}
```

**前端原实现**（Layout.tsx L55-56，已修复）：

```typescript
// ❌ 修复前：前端自己从 items 中过滤计算，且字段名不匹配
const items = (response?.data ?? response)?.items || [];
setNotificationCount(items.filter((item: { readAt?: string; read_at?: string }) => !item.readAt && !item.read_at).length);

// ✅ 修复后：直接使用后端返回的 unreadCount
const data = response?.data ?? response;
const count = data?.unreadCount ?? 0;
setNotificationCount(count);
```

| 字段 | 后端返回 | 前端原使用 | 匹配 | 修复后 |
|------|---------|-----------|------|--------|
| `unreadCount` | `number` | 未使用（前端自行计算） | ❌ | ✅ 直接使用 |
| `items[].readAt` | `string \| null`（驼峰） | `readAt` + `read_at` 双写 | ⚠️ | ✅ 不再依赖 |
| `items[].isRead` | `boolean` | 未使用 | — | ✅ 不再依赖 |

---

## 3️⃣ 数据结构一致性

### NavItem 类型

| 字段 | 任务卡定义 | 代码实现 | 一致性 |
|------|-----------|---------|--------|
| `key` | `string` | `string` | ✅ |
| `label` | `string` | `string` | ✅ |
| `icon` | `string` | `string` | ✅ |
| `path` | `string` | `string` | ✅ |
| `badge` | `number?` | `number?` | ✅ |
| `requiredRole` | 未定义 | `('user'\|'developer'\|'admin')[]?` | ✅ 扩展合理 |
| `children` | `NavItem[]?` | `NavItem[]?` | ✅ |

### RouteMapping 类型

| 字段 | 任务卡定义 | 代码实现 | 一致性 |
|------|-----------|---------|--------|
| `from` | `oldPath` | `from` | ⚠️ 命名不同 |
| `to` | `newPath` | `to` | ⚠️ 命名不同 |
| `redirectType` | `'301'\|'302'` | `status: 302` | ⚠️ 类型不同 |

**说明**：RouteMapping 字段命名与任务卡不一致，但代码实现更符合 HTTP 语义（`from/to/status`），且 `ROUTE_REDIRECTS` 仅作为文档配置，未在运行时使用（App.tsx 中直接硬编码了 `<Navigate>` 组件），不影响功能。

### User 类型（Layout 中使用）

| 字段 | 后端 auth/login 返回 | 前端 Layout 使用 | 一致性 |
|------|---------------------|-----------------|--------|
| `displayName` | ✅ 有（L746） | `user?.displayName` | ✅ |
| `tenantName` | ❌ 无 | `user?.tenantName` | ⚠️ 后端不返回此字段 |
| `email` | ❌ login 不返回 | `user?.email` | ⚠️ login 不返回 |

**说明**：`user?.tenantName` 和 `user?.email` 在 login 响应中不存在，但使用了 `||` 降级，最终会 fallback 到 `'用户'`，不影响功能。如需精确显示，应调用 `userApi.account.profile.get()` 获取完整信息。

---

## 4️⃣ 状态流一致性

T024 为导航重构任务，不涉及业务状态流转（如 Job 状态机）。导航相关状态：

| 状态 | 前端定义 | 后端对应 | 一致性 |
|------|---------|---------|--------|
| `activeNavKey` | 前端本地计算 | 无 | ✅ 纯前端状态 |
| `notificationCount` | 前端本地 state | 后端 `unreadCount` | ✅ 修复后一致 |
| `isMobile` | 前端本地 state | 无 | ✅ 纯前端状态 |
| `mobileDrawerOpen` | 前端本地 state | 无 | ✅ 纯前端状态 |

---

## 5️⃣ 错误处理

| 场景 | 处理方式 | 位置 | 评估 |
|------|---------|------|------|
| 通知 API 失败 | `.catch(() => undefined)` — 静默失败，保持 count=0 | Layout.tsx L58 | ✅ 合理（导航不应因通知失败而阻塞） |
| 通知 API 无数据 | `unreadCount ?? 0` — fallback 为 0 | Layout.tsx L55 | ✅ 合理 |
| 通知 API loading | 无 loading 状态 | Layout.tsx | ✅ 合理（导航徽章无需 loading 态） |
| 旧路由访问 | `<Navigate replace />` — 302 重定向 | App.tsx L57-66 | ✅ 合理 |
| 404 路由 | `<Navigate to={ROUTES.USER.HOME} replace />` | App.tsx L68 | ✅ 合理 |
| 未认证访问 | `<Navigate to={ROUTES.USER.LOGIN} />` | App.tsx L22 | ✅ 合理 |

---

## 6️⃣ Business Services 层检查

| API 调用 | 是否通过 Service 层 | Service 层方法 | 评估 |
|---------|-------------------|---------------|------|
| 通知列表 | ✅ 是 | `userApi.notifications.list()` | 通过 `@xbis/shared` 的 `apiClient` |
| 认证状态 | ✅ 是 | `useAuthStore()` | 通过 `@xbis/shared` 的 Zustand store |
| 路由跳转 | N/A | `navigate()` | 纯前端路由，无需 Service 层 |

**未绕过 Business Services 层** ✅

---

## 🧪 联调结果（D1）

👉 状态：**联调通过**（1 项前端已修复）

---

## ❌ 问题列表

| 类型 | 问题 | 位置 | 影响 | 状态 |
|------|------|------|------|------|
| 返回结构 | 前端自行从 items 过滤计算未读数，未使用后端 `unreadCount` 字段 | Layout.tsx L55-56 | 未读数可能不准确（`read_at` 字段不存在于新版 API） | ✅ 已修复 |
| 数据结构 | `RouteMapping` 字段命名与任务卡不一致（`from` vs `oldPath`） | types/index.ts L1219 | 不影响运行，仅文档差异 | ⚠️ 可接受 |
| 数据结构 | `user?.tenantName` 在 login 响应中不存在 | Layout.tsx L126 | UserDropdown 显示名称可能 fallback 到 `'用户'` | ⚠️ 低影响 |

---

## 🛠 修改建议

### 前端修改

1. **已修复**：Layout.tsx 通知未读数改为直接使用 `unreadCount` 字段
2. **建议优化**：UserDropdown 的 `name` 属性可改为调用 `userApi.account.profile.get()` 获取完整用户信息，确保 `displayName` 准确

### 后端修改

无。后端 API 返回结构完整、规范。

### 数据结构调整

无。现有数据结构满足需求。

---

## ⚠️ 风险评估

| 风险项 | 等级 | 说明 |
|-------|------|------|
| 通知未读数准确性 | 已修复 | 改为使用后端 `unreadCount`，不再前端计算 |
| UserDropdown 显示名 | 低 | login 响应不含 `tenantName`/`email`，但 fallback 逻辑保证不报错 |
| 后续任务影响 | 无 | T024 为纯导航重构，不修改数据模型 |

---

## ✅ 是否允许进入验收（D2）

👉 **YES**

所有 API 契约一致，返回结构匹配，数据结构一致，错误处理完善，Business Services 层合规。唯一的前端问题已修复。
