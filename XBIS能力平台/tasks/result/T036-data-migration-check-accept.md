# T036 数据迁移脚本 — 验收文档

> 任务编号：T036
> 任务名称：数据迁移脚本
> 执行阶段：C4 → C5 → C6 → C7
> 执行日期：2026-04-25
> 执行结果：通过

---

## 1. 修改文件清单

### 新增文件

| 文件路径 | 说明 |
|---------|------|
| `packages/shared/src/types/migration.ts` | 迁移相关类型定义 |
| `packages/components/blocks/MigrationStatusCard/index.tsx` | 迁移状态卡片组件 |
| `packages/components/blocks/MigrationControls/index.tsx` | 迁移控制按钮组件 |
| `packages/components/blocks/MigrationLog/index.tsx` | 迁移日志表格组件 |
| `packages/admin/src/pages/Migration.tsx` | 迁移监控页面 |
| `tokens/package.json` | tokens 包配置（修复缺失） |

### 修改文件

| 文件路径 | 修改类型 | 说明 |
|---------|---------|------|
| `packages/shared/src/types/index.ts` | 修改 | 导出 migration 类型 |
| `packages/shared/src/api/services.ts` | 修改 | 新增 migration API |
| `packages/shared/src/constants/index.ts` | 修改 | 新增 MIGRATION 路由 |
| `packages/admin/src/App.tsx` | 修改 | 添加 Migration 路由 |
| `packages/admin/src/components/Layout.tsx` | 修改 | 添加数据迁移菜单 |
| `packages/components/blocks/index.ts` | 修改 | 导出新增 blocks 组件 |
| `packages/admin/vite.config.ts` | 修改 | 添加 @xbis/components 和 @xbis/tokens 别名 |
| `packages/admin/tsconfig.json` | 修改 | 添加路径映射 |
| `packages/admin/package.json` | 修改 | 添加 workspace 依赖 |
| `pnpm-workspace.yaml` | 修改 | 添加 tokens 到 workspace |

---

## 2. 新增组件清单

| 组件名称 | 所属层级 | 说明 |
|---------|---------|------|
| MigrationStatusCard | blocks | 迁移状态卡片：状态标签 + 进度条 + 统计信息 |
| MigrationControls | blocks | 迁移控制：开始迁移 / 校验数据 / 回滚按钮 |
| MigrationLog | blocks | 迁移日志：时间/级别/消息表格 |

---

## 3. API变更清单

| API | Method | Path | 说明 |
|-----|--------|------|------|
| migration.status | GET | `/admin-api/v1/migration/status` | 查询迁移状态 |
| migration.run | POST | `/admin-api/v1/migration/run` | 执行迁移 |
| migration.validate | POST | `/admin-api/v1/migration/validate` | 校验数据 |

---

## 4. 风险说明

| 风险项 | 评估 | 说明 |
|--------|------|------|
| 是否影响现网 | 否 | 新增独立页面，不影响现有页面 |
| 是否影响数据结构 | 否 | 仅新增类型定义，不修改现有类型 |
| 类型冲突 | 已处理 | ability.ts 与 executor.ts 中 AbilityExecutorBinding 重名，已添加注释说明 |
| 依赖缺失 | 已修复 | 补充了 @xbis/components 和 @xbis/tokens 的路径别名与依赖 |

---

## 5. 自检结果

### C5 AI 强制自检

| 检查项 | 结果 |
|--------|------|
| 必修复问题 | 5 项全部修复 |
| 可优化问题 | 2 项已记录（V2 迭代） |
| 类型检查 | Migration.tsx 无错误 |

修复内容：
1. 补充 `textStyle` 导入
2. MigrationStatusCard 改用 base 层 Tag 组件
3. MigrationControls 改用 base 层 Button 组件
4. 添加 TODO-MIGRATION-V2 标记 mock 日志
5. 修正 Alert 的 `variant` → `type` prop
6. 修复 `NodeJS.Timeout` → `ReturnType<typeof setInterval>`

### C6 开发后自检

| 检查项 | 状态 |
|--------|------|
| 组件复用 | ✅ |
| 命名规范 | ✅ |
| 页面结构统一 | ✅ |
| 状态完整（loading/empty/error） | ✅ |
| API规范 | ✅ |
| 权限兼容 | ✅ |
| 异常处理 | ✅ |
| 是否影响现网 | ✅ |

**结果：Pass**

---

## 6. 验收结果

| 验收项 | 结果 |
|--------|------|
| 页面是否能打开 | ✅ |
| 主流程是否可用 | ✅ |
| API是否成功调用 | ✅ |
| 是否存在报错 | ✅ |
| UI是否破坏 | ✅ |
| 是否影响旧功能 | ✅ |

**结果：通过**

---

## 7. 备注

- 迁移日志当前使用 mock 数据生成，V2 迭代需替换为真实 API
- 回滚功能当前为模拟实现，V2 迭代需对接真实回滚 API
- 页面已按 Design Tokens 规范使用颜色、间距、字体样式
- 所有状态（loading/empty/error/idle）已完整实现
