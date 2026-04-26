# T032 管理端导航重构 — 设计评审报告

> 评审日期: 2026-04-24  
> 评审人: 技术负责人 + 架构审查官  
> 状态: Passed with changes

---

## 评审结果

### 状态: 👉 Passed with changes

---

## 问题列表

### 问题1: 旧路由映射表未明确
**严重程度: 中**

设计方案提到"302 重定向到新路由"，但未列出具体的旧路由和新路由映射表。

### 问题2: 导航权限控制未细化
**严重程度: 中**

NavMenu 支持权限控制，但未说明不同角色（admin/superadmin/operator）能看到哪些菜单项。

### 问题3: 缺少导航项的徽章/通知支持
**严重程度: 低**

管理端导航可能需要显示徽章（如待审核任务数、告警数），但设计方案未提及。

### 问题4: 全局搜索（V2）占位未说明
**严重程度: 低**

AdminHeader 中提到"全局搜索（V2）"，但未说明 V1 版本如何处理。

---

## 修改建议

### 建议1: 明确旧路由映射表（必须修改）
```typescript
const adminRouteRedirects = [
  { from: '/admin/reviews', to: '/admin/operations', status: 302 },
  { from: '/admin/settings/old', to: '/admin/settings', status: 302 },
  // ... 其他映射
];
```

### 建议2: 细化导航权限配置（必须修改）
```typescript
interface AdminNavItem {
  key: string;
  label: string;
  path: string;
  requiredRole: ('admin' | 'superadmin' | 'operator')[];
  badge?: 'alerts' | 'pendingTasks';
}

const navPermissions = {
  overview: ['admin', 'superadmin', 'operator'],
  abilities: ['admin', 'superadmin'],
  executors: ['admin', 'superadmin'],
  operations: ['admin', 'superadmin', 'operator'],
  users: ['admin', 'superadmin'],
  billing: ['admin', 'superadmin'],
  settings: ['admin', 'superadmin'],
  logs: ['admin', 'superadmin'],
  profile: ['admin', 'superadmin', 'operator']
};
```

### 建议3: 增加导航徽章支持（建议修改）
- 概览：显示告警数量
- 任务运营：显示待处理任务数
- 全局使用通知 Badge 组件

### 建议4: 明确 V1 搜索方案（建议修改）
- V1：隐藏全局搜索占位符
- V2：实现全局搜索功能

---

## 是否允许进入开发

### 👉 YES（附条件通过）

**必须完成的条件**:
1. 明确旧路由映射表（建议1）
2. 细化导航权限配置（建议2）

---

## 评审总结

| 检查项 | 结果 | 说明 |
|--------|------|------|
| 组件复用原则 | ✅ | 复用 NavMenu |
| 页面模板使用 | ✅ | 使用 AdminLayout |
| 重复组件 | ✅ | 无重复 |
| 分层规范 | ✅ | 符合分层 |
| API 合理性 | N/A | 无 API |
| 状态设计完整性 | ✅ | 状态完整 |
| 能力平台架构 | ✅ | 符合 |
| 过度设计 | ✅ | 不过度 |
| 未声明数据模型 | ⚠️ | 权限映射未定义 |

**风险等级**: 低
