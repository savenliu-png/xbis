# T036 数据迁移脚本 — 设计评审报告

> 评审日期: 2026-04-24  
> 评审人: 技术负责人 + 架构审查官  
> 状态: Passed with changes

---

## 评审结果

### 状态: 👉 Passed with changes

---

## 问题列表

### 问题1: 迁移脚本缺少数据校验的详细规则
**严重程度: 高**

数据迁移后需要校验数据一致性，但设计方案未说明具体的校验规则（如记录数对比、字段值对比、关联关系校验等）。

### 问题2: 回滚脚本的范围和限制未明确
**严重程度: 高**

回滚脚本可以恢复数据，但如果迁移后已有新数据写入，回滚可能导致数据丢失。设计方案未说明回滚的限制条件。

### 问题3: 迁移过程中的服务可用性未考虑
**严重程度: 中**

数据迁移期间，现网服务是否可用？如果可用，如何处理迁移过程中的数据写入？

### 问题4: 缺少迁移前的数据备份要求
**严重程度: 高**

数据迁移是高风险操作，必须在迁移前备份数据，但设计方案未明确备份要求。

---

## 修改建议

### 建议1: 详细设计数据校验规则（必须修改）
```typescript
interface MigrationValidationRule {
  name: string;
  sourceQuery: string;
  targetQuery: string;
  comparator: 'count' | 'sum' | 'exact';
}

const validationRules: MigrationValidationRule[] = [
  { name: '记录数一致', sourceQuery: 'SELECT COUNT(*) FROM api_market_item', targetQuery: 'SELECT COUNT(*) FROM abilities', comparator: 'count' },
  { name: '字段值一致', sourceQuery: 'SELECT id, name FROM api_market_item', targetQuery: 'SELECT id, display_name FROM abilities', comparator: 'exact' }
];
```

### 建议2: 明确回滚限制（必须修改）
```typescript
interface RollbackLimitation {
  maxRollbackTime: number; // 迁移后多长时间内可回滚（如 24 小时）
  allowNewData: boolean;   // 是否允许迁移后有新数据写入
  backupRequired: boolean; // 是否要求备份
}
```

### 建议3: 设计迁移期间的服务策略（必须修改）
- **方案A（停机迁移）**: 迁移期间服务不可用，显示维护页面
- **方案B（在线迁移）**: 双写机制，迁移期间同时写入旧表和新表
- 推荐方案A（简单可控），如业务要求 0 停机则采用方案B

### 建议4: 强制数据备份（必须修改）
迁移前必须：
1. 全量备份源数据库
2. 备份目标数据库（如已有数据）
3. 记录备份时间点和位置
4. 验证备份可恢复性

---

## 是否允许进入开发

### 👉 YES（附条件通过）

**必须完成的条件**:
1. 详细设计数据校验规则（建议1）
2. 明确回滚限制（建议2）
3. 设计迁移期间的服务策略（建议3）
4. 强制数据备份（建议4）

---

## 评审总结

| 检查项 | 结果 | 说明 |
|--------|------|------|
| 组件复用原则 | ✅ | 复用 Progress、Table |
| 页面模板使用 | ✅ | 使用 AdminPageShell |
| 重复组件 | ✅ | 无重复 |
| 分层规范 | ✅ | 符合分层 |
| API 合理性 | ✅ | API 定义清晰 |
| 状态设计完整性 | ✅ | 状态完整 |
| 能力平台架构 | ✅ | 符合 |
| 过度设计 | ✅ | 不过度 |
| 未声明数据模型 | ⚠️ | 校验规则未定义 |

**风险等级**: 高
