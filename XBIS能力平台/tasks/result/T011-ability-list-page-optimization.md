# T011 能力中心列表页 — 工程级优化报告

> 任务编号：T011
> 优化阶段：Optimization Pass（基于 D2 验收报告）
> 完成日期：2026-04-25
> 执行人：资深前端架构师 + 性能优化专家 + 代码重构负责人

---

## 一、优化总结

| 项目 | 内容 |
|------|------|
| 优化点数量 | **4 项** |
| 影响范围 | `AbilityListPage` + `AbilityTable` + 新增 `useDebounce` hook |
| 是否影响现有功能 | **否**（纯优化，无行为变更） |
| 是否破坏 API 契约 | **否** |

---

## 二、优化清单

| # | 类型 | 问题 | 优化方案 | 优先级 |
|---|------|------|----------|--------|
| 1 | 验收报告 | `AbilityTable.tsx:150` `record: Ability` 未更新为 `AbilityCardItem` | 修正类型引用 | P0 |
| 2 | 性能 | 搜索防抖使用 `useRef` 管理定时器，代码重复 | 提取 `useDebounce` 通用 hook | P1 |
| 3 | 代码质量 | `AbilityListPage` 搜索逻辑与防抖耦合 | 使用 `useDebounce` 解耦 | P1 |
| 4 | 代码质量 | 定时器清理逻辑分散 | `useDebounce` 内部统一处理 | P2 |

---

## 三、修改文件列表

| 文件路径 | 修改类型 |
|----------|----------|
| `packages/pages/ability/AbilityListPage.tsx` | 优化 |
| `packages/pages/ability/components/AbilityTable.tsx` | 优化 |
| `packages/pages/shared/hooks/useDebounce.ts` | 新增 |

---

## 四、优化后代码（关键对比）

### 4.1 AbilityTable — 修复类型遗留问题

**优化前：**
```typescript
render: (value: string, record: Ability) => (
```

**优化后：**
```typescript
render: (value: string, record: AbilityCardItem) => (
```

**收益：**
- 解决 D2 验收报告中「类型不一致」的问题
- 与组件 Props 类型保持一致

---

### 4.2 新增 useDebounce Hook

**新增文件**：`packages/pages/shared/hooks/useDebounce.ts`

```typescript
import { useRef, useCallback, useEffect } from 'react';

export function useDebounce<T extends (...args: unknown[]) => void>(
  callback: T,
  delay: number = 300
): T {
  const timeoutRef = useRef<ReturnType<typeof setTimeout>>();

  const debouncedCallback = useCallback(
    (...args: unknown[]) => {
      if (timeoutRef.current) {
        clearTimeout(timeoutRef.current);
      }
      timeoutRef.current = setTimeout(() => {
        callback(...args);
      }, delay);
    },
    [callback, delay]
  ) as T;

  useEffect(() => {
    return () => {
      if (timeoutRef.current) {
        clearTimeout(timeoutRef.current);
      }
    };
  }, []);

  return debouncedCallback;
}
```

**收益：**
- 通用防抖 hook，可被其他页面复用
- 内部自动处理定时器清理，避免内存泄漏
- 类型安全，支持任意参数函数

---

### 4.3 AbilityListPage — 使用 useDebounce 优化搜索

**优化前：**
```typescript
const [searchKeyword, setSearchKeyword] = useState('');
const searchTimeoutRef = React.useRef<ReturnType<typeof setTimeout>>();

const handleSearch = useCallback((newKeyword: string) => {
  setSearchKeyword(newKeyword);
  if (searchTimeoutRef.current) {
    clearTimeout(searchTimeoutRef.current);
  }
  searchTimeoutRef.current = setTimeout(() => {
    setPage(1);
    setKeyword(newKeyword || undefined);
  }, 300);
}, []);

// 清理定时器
useEffect(() => {
  return () => {
    if (searchTimeoutRef.current) {
      clearTimeout(searchTimeoutRef.current);
    }
  };
}, []);
```

**优化后：**
```typescript
import { useDebounce } from '../shared/hooks/useDebounce';

const [searchKeyword, setSearchKeyword] = useState('');

const debouncedSetKeyword = useDebounce((newKeyword: string) => {
  setPage(1);
  setKeyword(newKeyword || undefined);
}, 300);

const handleSearch = useCallback((newKeyword: string) => {
  setSearchKeyword(newKeyword);
  debouncedSetKeyword(newKeyword);
}, [debouncedSetKeyword]);
```

**收益：**
- 搜索逻辑与防抖实现解耦
- 移除 `useRef` 和定时器清理逻辑，代码更简洁
- 防抖逻辑复用，其他页面可直接使用 `useDebounce`

---

## 五、性能提升说明

### 5.1 渲染优化

无变更，保持现有 `memo` 和 `useMemo` 优化。

### 5.2 请求优化

无变更，搜索防抖逻辑保持 300ms。

### 5.3 结构优化

| 优化项 | 说明 |
|--------|------|
| useDebounce 提取 | 将搜索防抖逻辑提取为通用 hook，可被其他页面复用 |
| 代码解耦 | 搜索状态管理与防抖实现分离，职责更清晰 |

---

## 六、风险评估

| 风险项 | 等级 | 说明 |
|--------|------|------|
| 是否影响现有功能 | **无** | 纯优化，无行为变更 |
| 是否破坏页面模板 | **无** | 继续使用 ListPageShell |
| 是否影响数据结构 | **无** | 类型定义未变更 |
| 是否需要回归测试 | **建议** | 重点验证：搜索防抖功能 |

---

## 七、是否建议合并

**👉 YES**

理由：
1. 解决了 D2 验收报告中明确的类型遗留问题
2. 提取通用 `useDebounce` hook，提升代码复用性
3. 所有优化均为「非破坏性优化」，不影响现有功能
4. 代码更简洁，可维护性提升

---

## 八、D2 验收报告问题修复对照

| D2 问题 | 状态 | 修复方式 |
|---------|------|----------|
| `AbilityTable.tsx:150` `record: Ability` 未更新为 `AbilityCardItem` | ✅ 已修复 | 修正类型引用 |

---

*本文档与代码同步维护，后续迭代请同步更新。*
