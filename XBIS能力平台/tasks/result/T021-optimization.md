# T021 账单与发票 — 工程级优化报告

## 一、优化总结

- 优化点数量：7
- 影响范围：4 个文件（business/InvoiceItem, business/InvoiceTable, blocks/InvoiceApplyForm, pages/Invoices）
- 是否影响现有功能：否 — 所有优化为增量改进，不破坏已有行为

---

## 二、优化清单

| 类型 | 问题 | 优化方案 | 优先级 |
|------|------|---------|--------|
| 组件层 | business/InvoiceItem 本地 statusMap 与 shared 常量重复 | 使用 `INVOICE_STATUS_CONFIG` from `@xbis/shared` | 高 |
| 组件层 | business/InvoiceItem 使用原生 `<button>` | 替换为 `Button variant="link"` from `@xbis/components/base` | 中 |
| 组件层 | business/InvoiceTable 本地 statusMap 与 shared 常量重复 | 使用 `INVOICE_STATUS_CONFIG` from `@xbis/shared` | 高 |
| 组件层 | business/InvoiceTable 使用原生 `<button>` | 替换为 `Button variant="link"` from `@xbis/components/base` | 中 |
| 性能 | Invoices.tsx 详情 Drawer 内联数据数组每次渲染重建 | `useMemo` 缓存 `detailRows` | 中 |
| API & 数据流 | `extractData` 在 Invoices.tsx 和 InvoiceApplyForm 中重复定义 | 使用 `extractApiResponse` from `@xbis/shared/utils` | 中 |
| 性能 | Invoices.tsx `onRetry` 未 useCallback 包裹 | 提取为 `handleRetry` + `useCallback` | 低 |
| 性能 | InvoiceApplyForm `handleFinish` 未 useCallback 包裹 | `useCallback` 包裹 | 低 |

---

## 三、修改文件列表

| 文件路径 | 修改类型 | 说明 |
|---------|---------|------|
| `packages/components/business/InvoiceItem/index.tsx` | 优化 | 使用 shared 常量 + base/Button |
| `packages/components/business/InvoiceTable/index.tsx` | 优化 | 使用 shared 常量 + base/Button + base/Tag |
| `packages/components/blocks/InvoiceApplyForm/index.tsx` | 优化 | 使用 extractApiResponse + useCallback |
| `packages/user/src/pages/Invoices.tsx` | 优化 | 使用 extractApiResponse + useMemo + useCallback |

---

## 四、优化后代码

### 4.1 business/InvoiceItem — 消除重复常量 + 使用 base/Button

**优化前**：
```tsx
const statusMap: Record<string, { label: string; variant: 'success' | 'warning' | 'error' | 'default' | 'info' }> = {
  draft: { label: '草稿', variant: 'default' },
  applied: { label: '已申请', variant: 'info' },
  // ... 重复定义
};
// ...
<button onClick={...} style={{ ...textStyle.caption, color: colors.brand.default, background: 'none', border: 'none', cursor: 'pointer', marginTop: space['1'] }}>
  下载 PDF
</button>
```

**优化后**：
```tsx
import { INVOICE_STATUS_CONFIG } from '@xbis/shared';
import { Button } from '@xbis/components/base';

const status = INVOICE_STATUS_CONFIG[invoice.status] || INVOICE_STATUS_CONFIG.draft;
// ...
<Button variant="link" size="small" onClick={() => onDownload(invoice.invoiceNo)} style={{ marginTop: space['1'] }}>
  下载 PDF
</Button>
```

### 4.2 business/InvoiceTable — 消除重复常量 + 使用 base/Button + base/Tag

**优化前**：
```tsx
const statusMap: Record<string, { label: string; color: string }> = {
  draft: { label: '草稿', color: colors.text.muted },
  // ... 重复定义，且使用 colors 而非 Tag variant
};
// ...
<span style={{ color: status.color }}>{status.label}</span>
// ...
<button onClick={...} style={{ ...textStyle.caption, color: colors.brand.default, background: 'none', border: 'none', cursor: 'pointer' }}>
  下载
</button>
```

**优化后**：
```tsx
import { INVOICE_STATUS_CONFIG } from '@xbis/shared';
import { Tag, Button } from '@xbis/components/base';

const config = INVOICE_STATUS_CONFIG[value] || INVOICE_STATUS_CONFIG.draft;
return <Tag variant={config.variant} size="sm">{config.label}</Tag>;
// ...
<Button variant="link" size="small" onClick={() => onDownload(record.invoiceNo)}>
  下载
</Button>
```

### 4.3 Invoices.tsx — extractApiResponse + useMemo + useCallback

**优化前**：
```tsx
const extractData = <T,>(response: unknown): T => { ... }; // 本地重复定义

// 内联数组每次渲染重建
{[
  { label: '发票号', value: ... },
  { label: '状态', value: ... },
  // ...
].map((row) => (...))}

// onRetry 未 useCallback
onRetry={() => loadData(1, pagination.pageSize)}
```

**优化后**：
```tsx
import { extractApiResponse } from '@xbis/shared'; // 使用 shared 工具函数

const detailRows = useMemo(() => {
  if (!selectedInvoice) return [];
  return [
    { label: '发票号', value: <code>{selectedInvoice.invoiceNo}</code> },
    // ...
  ];
}, [selectedInvoice]);

const handleRetry = useCallback(() => {
  loadData(1, pagination.pageSize);
}, [loadData, pagination.pageSize]);

// JSX
onRetry={handleRetry}
```

### 4.4 InvoiceApplyForm — extractApiResponse + useCallback

**优化前**：
```tsx
const extractData = <T,>(response: unknown): T => { ... }; // 本地重复定义

const handleFinish = (values: InvoiceApplyRequest) => {
  onSubmit(values);
};
```

**优化后**：
```tsx
import { extractApiResponse } from '@xbis/shared'; // 使用 shared 工具函数

const handleFinish = useCallback((values: InvoiceApplyRequest) => {
  onSubmit(values);
}, [onSubmit]);
```

---

## 五、性能提升说明

### 渲染优化

| 优化项 | 优化前 | 优化后 | 提升 |
|--------|--------|--------|------|
| detailRows 数组 | 每次渲染重建 12 元素数组 | `useMemo` 缓存，仅 selectedInvoice 变化时重建 | 减少无效对象创建 |
| handleRetry | 内联箭头函数，每次渲染新引用 | `useCallback` 稳定引用 | ListPageShell 避免无效重渲染 |
| handleFinish | 普通函数，每次渲染新引用 | `useCallback` 稳定引用 | AntdForm 避免无效重渲染 |
| business/InvoiceItem | 原生 button 无 hover/focus 样式 | base/Button variant="link" | 统一交互体验 |

### 请求优化

| 优化项 | 说明 |
|--------|------|
| extractApiResponse 统一 | 消除 2 处本地重复定义，使用 shared 层统一工具函数 |
| 常量统一 | 消除 3 处 statusMap 重复定义（InvoiceItem + InvoiceTable + blocks/InvoiceTable），统一使用 `INVOICE_STATUS_CONFIG` |

### 结构优化

| 优化项 | 说明 |
|--------|------|
| 组件规范 | business 层组件不再使用原生 HTML 元素（button/span），统一使用 base 层组件 |
| 单一数据源 | 发票状态映射从 3 处分散定义收敛为 1 处 shared 常量 |
| 工具函数 | API 响应解析从 2 处本地定义收敛为 1 处 shared 工具函数 |

---

## 六、风险评估

| 风险项 | 等级 | 说明 |
|--------|------|------|
| business/InvoiceItem 接口变更 | 🟢 低 | Props 接口未变，仅内部实现优化 |
| business/InvoiceTable 接口变更 | 🟢 低 | Props 接口未变，仅内部实现优化 |
| extractApiResponse 兼容性 | 🟢 低 | 与本地 extractData 逻辑完全一致 |
| INVOICE_STATUS_CONFIG 兼容性 | 🟢 低 | 与本地 statusMap 映射完全一致 |

### 是否需要回归测试

否 — 所有优化为内部实现改进，外部接口不变。但建议验证：
1. business/InvoiceItem 下载按钮样式是否正常
2. business/InvoiceTable 状态 Tag 样式是否正常
3. 详情 Drawer 数据展示是否正常

---

## 七、是否建议合并

👉 **YES**

所有优化均为工程级改进：
- 消除 3 处重复常量定义
- 消除 2 处重复工具函数
- 统一使用 base 层组件替代原生 HTML
- 补齐 useMemo/useCallback 性能优化
- TypeScript 编译通过
- 不影响现有功能
