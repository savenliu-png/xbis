# XBIS 页面模板设计规范

> 版本：v1.0  
> 设计原则：Desktop-first（优先桌面端 ≥1200px）  
> 适用范围：XBIS 用户端 + 管理端

---

## 1. 设计原则

### 1.1 Desktop-first
- 所有页面优先设计桌面端布局（≥1200px）
- 页面结构必须支持多栏布局（侧边栏 + 主区 + 操作区）
- 移动端适配方案作为降级策略，允许裁剪功能
- 管理端无需移动端设计

### 1.2 页面模板分层
```
┌─────────────────────────────────────────────────────────────┐
│  Page Shell (页面骨架)                                       │
│  ─────────────────────────────────────────────────────────  │
│  负责：页面级布局、状态管理（空/加载/错误）、响应式适配        │
├─────────────────────────────────────────────────────────────┤
│  Page Blocks (页面块)                                        │
│  ─────────────────────────────────────────────────────────  │
│  负责：页面内独立功能区块（统计卡片、筛选栏、数据表格等）       │
├─────────────────────────────────────────────────────────────┤
│  Business Components (业务组件)                              │
│  ─────────────────────────────────────────────────────────  │
│  负责：业务实体展示（能力卡片、任务项、账单卡片等）             │
├─────────────────────────────────────────────────────────────┤
│  Base Components (基础组件)                                  │
│  ─────────────────────────────────────────────────────────  │
│  负责：UI 原子元素（按钮、输入框、卡片、标签等）               │
└─────────────────────────────────────────────────────────────┘
```

### 1.3 状态管理
每个页面模板必须支持四种状态：
- **idle**：正常展示数据
- **loading**：数据加载中
- **empty**：数据为空
- **error**：加载出错

---

## 2. 页面模板总览

| 模板 | 类型 | 适用场景 | 文件位置 |
|------|------|----------|----------|
| ListPageShell | 通用 | 用户列表、调用记录、发票列表、审核列表 | `layout/ListPageShell` |
| DetailPageShell | 通用 | 能力详情、任务详情、用户详情 | `layout/DetailPageShell` |
| FormPageShell | 通用 | 创建/编辑表单、配置页面 | `layout/FormPageShell` |
| TaskPageShell | **一级模板** | 任务中心（列表/详情/创建/监控） | `layout/TaskPageShell` |
| AbilityPageShell | **一级模板** | 能力中心（网格/详情/测试/接入） | `layout/AbilityPageShell` |

---

## 3. 列表页模板（ListPageShell）

### 3.1 页面结构

```
┌─────────────────────────────────────────────────────────────┐
│  PageHeader                                                 │
│  ├── 标题 + 副标题                                          │
│  ├── 面包屑导航                                             │
│  └── 操作按钮（新建/导出/刷新）                              │
├─────────────────────────────────────────────────────────────┤
│  Filter Bar（筛选栏）                                       │
│  ├── 搜索框                                                 │
│  ├── 状态筛选标签                                           │
│  └── 高级筛选器（可选）                                      │
├─────────────────────────────────────────────────────────────┤
│  Batch Action Bar（批量操作栏，选中时显示）                   │
│  └── 批量通过 / 批量拒绝 / 批量删除 / 导出                   │
├─────────────────────────────────────────────────────────────┤
│  Data Area（数据区）                                        │
│  ├── 状态: loading → Spinner                                │
│  ├── 状态: empty → Empty                                    │
│  ├── 状态: error → Alert + 重试按钮                          │
│  └── 状态: idle → Table / Card List                         │
├─────────────────────────────────────────────────────────────┤
│  Pagination（分页器）                                       │
└─────────────────────────────────────────────────────────────┘
```

### 3.2 组件组成

| 区域 | 使用组件 | 来源 |
|------|----------|------|
| 页面标题 | PageHeader | `layout/PageHeader` |
| 筛选栏 | Search, Tag, Select | `base/Search`, `base/Tag` |
| 批量操作 | AuditActionBar | `business/AuditActionBar` |
| 数据表格 | Table | `base/Table` |
| 卡片列表 | Card + business 组件 | `base/Card` + `business/*` |
| 分页 | Pagination | Ant Design |
| 空态 | Empty | `base/Empty` |
| 加载 | Spinner | `base/Spinner` |
| 错误 | Alert | `base/Alert` |

### 3.3 数据流

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│  Filter Bar │────▶│  API Call   │────▶│  List State │
│  (用户输入)  │     │  (数据获取)  │     │  (数据存储)  │
└─────────────┘     └─────────────┘     └──────┬──────┘
                                               │
                         ┌─────────────────────┼─────────────────────┐
                         ▼                     ▼                     ▼
                   ┌──────────┐          ┌──────────┐          ┌──────────┐
                   │  loading │          │  empty   │          │   idle   │
                   │  Spinner │          │  Empty   │          │  Table   │
                   └──────────┘          └──────────┘          └──────────┘
                         ▲                     ▲                     ▲
                         │                     │                     │
                    ┌────┴────┐           ┌────┴────┐           ┌────┴────┐
                    │  error  │           │  error  │           │  error  │
                    │  Alert  │           │  Alert  │           │  Alert  │
                    └─────────┘           └─────────┘           └─────────┘
```

### 3.4 状态展示

#### 空态
- 图标：Inbox 风格
- 标题："暂无数据"
- 描述：根据页面上下文定制，如 "还没有创建任何 API 密钥"
- 操作：提供新建按钮（如有权限）

#### 加载态
- 全区域居中 Spinner
- 提示文字："加载中..."
- 表格模式：显示 Skeleton 行占位

#### 错误态
- Alert 组件，variant="error"
- 显示错误信息
- 提供"重试"按钮
- 可选：提供"联系支持"链接

### 3.5 使用示例

```tsx
import { ListPageShell } from '@xbis/components/layout';
import { TaskFilterBar, ApiKeyTable } from '@xbis/components/blocks';

const ApiKeysPage = () => {
  const [status, setStatus] = useState<ListPageStatus>('idle');
  const [apiKeys, setApiKeys] = useState([]);

  return (
    <ListPageShell
      title="API 密钥管理"
      subtitle="管理您的 API 访问密钥"
      breadcrumbs={[{ label: '首页', path: '/' }, { label: '开发者中心' }, { label: 'API 密钥' }]}
      primaryAction={{ label: '新建密钥', icon: <PlusOutlined />, onClick: () => openCreateModal() }}
      filterBar={<TaskFilterBar statusOptions={[...]} onSearch={handleSearch} />}
      status={status}
      errorMessage={error?.message}
      onRetry={loadData}
      pagination={<Pagination total={100} />}
    >
      <ApiKeyTable apiKeys={apiKeys} onRevoke={handleRevoke} />
    </ListPageShell>
  );
};
```

---

## 4. 详情页模板（DetailPageShell）

### 4.1 页面结构

```
┌─────────────────────────────────────────────────────────────┐
│  PageHeader                                                 │
│  ├── 返回按钮 + 标题 + 副标题                                │
│  ├── 面包屑导航                                             │
│  └── 操作按钮（编辑/删除/导出）                              │
├─────────────────────────────────────────────────────────────┤
│  Tabs（可选，多标签详情）                                    │
├─────────────────────────────────────────────────────────────┤
│  Main Content Area                                          │
│  ├── 状态: loading → Spinner                                │
│  ├── 状态: notFound → Empty "内容不存在"                     │
│  ├── 状态: error → Alert + 重试按钮                          │
│  └── 状态: idle → 多栏布局                                   │
│       ├── 主信息区（70%）                                    │
│       │   ├── 基本信息卡片                                   │
│       │   ├── 详细数据表格/列表                              │
│       │   └── 时间线/日志                                    │
│       └── 侧边信息区（30%）                                  │
│           ├── 状态概览                                       │
│           ├── 快捷操作                                       │
│           └── 关联信息                                       │
└─────────────────────────────────────────────────────────────┘
```

### 4.2 组件组成

| 区域 | 使用组件 | 来源 |
|------|----------|------|
| 页面标题 | PageHeader | `layout/PageHeader` |
| 标签页 | Tabs | `base/Tabs` |
| 主信息区 | Card + Descriptions | `base/Card` |
| 侧边栏 | Card + DataMetric | `base/Card` + `business/DataMetric` |
| 时间线 | JobTimeline | `business/JobTimeline` |
| 状态徽章 | TaskStatusBadge | `business/TaskStatusBadge` |

### 4.3 数据流

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│  Route ID   │────▶│  API Call   │────▶│ Detail State│
│  (URL参数)   │     │  (数据获取)  │     │  (数据存储)  │
└─────────────┘     └─────────────┘     └──────┬──────┘
                                               │
                    ┌──────────────────────────┼──────────────────────────┐
                    ▼                          ▼                          ▼
              ┌──────────┐              ┌──────────┐              ┌──────────┐
              │ loading  │              │ notFound │              │   idle   │
              │ Spinner  │              │  Empty   │              │  Detail  │
              └──────────┘              └──────────┘              └──────────┘
                    ▲                          ▲                          ▲
                    │                          │                          │
               ┌────┴────┐                ┌────┴────┐                ┌────┴────┐
               │  error  │                │  error  │                │  error  │
               │  Alert  │                │  Alert  │                │  Alert  │
               └─────────┘                └─────────┘                └─────────┘
```

### 4.4 状态展示

#### 空态（notFound）
- 图标：文件搜索风格
- 标题："内容不存在或已被删除"
- 描述："请检查链接是否正确，或返回列表页"
- 操作："返回列表"按钮

#### 加载态
- 全区域居中 Spinner
- 提示文字："加载中..."
- 骨架屏：标题区 + 两栏卡片占位

#### 错误态
- Alert 组件，variant="error"
- 显示错误信息
- 提供"重试"按钮

### 4.5 使用示例

```tsx
import { DetailPageShell } from '@xbis/components/layout';
import { AbilityDetailPanel } from '@xbis/components/blocks';

const AbilityDetailPage = () => {
  const { apiId } = useParams();
  const [status, setStatus] = useState<DetailPageStatus>('loading');
  const [ability, setAbility] = useState(null);

  return (
    <DetailPageShell
      title={ability?.displayName || '能力详情'}
      subtitle={ability?.apiName}
      backPath="/abilities"
      breadcrumbs={[...]}
      primaryAction={{ label: '立即接入', onClick: () => subscribe() }}
      sidebar={<RelatedAbilities abilityId={apiId} />}
      status={status}
      errorMessage={error?.message}
      onRetry={loadData}
    >
      <AbilityDetailPanel ability={ability} />
    </DetailPageShell>
  );
};
```

---

## 5. 表单页模板（FormPageShell）

### 5.1 页面结构

```
┌─────────────────────────────────────────────────────────────┐
│  PageHeader                                                 │
│  ├── 返回按钮 + 标题 + 副标题                                │
│  ├── 面包屑导航                                             │
│  └── （表单页通常无顶部操作按钮）                             │
├─────────────────────────────────────────────────────────────┤
│  Form Content Area                                          │
│  ├── 状态: loading → Spinner                                │
│  ├── 状态: error → Alert + 重试按钮                          │
│  └── 状态: idle → 表单布局                                   │
│       ├── 表单主区（70%）                                    │
│       │   ├── 分组表单卡片                                   │
│       │   │   ├── 基本信息                                   │
│       │   │   ├── 配置参数                                   │
│       │   │   └── 高级选项                                   │
│       │   └── 底部操作栏                                     │
│       │       ├── 取消按钮                                   │
│       │       └── 保存按钮（loading 态）                      │
│       └── 辅助信息区（30%，可选）                             │
│           ├── 表单说明                                       │
│           ├── 字段帮助                                       │
│           └── 预览效果                                       │
└─────────────────────────────────────────────────────────────┘
```

### 5.2 组件组成

| 区域 | 使用组件 | 来源 |
|------|----------|------|
| 页面标题 | PageHeader | `layout/PageHeader` |
| 表单卡片 | Card + Form | `base/Card` + `base/Form` |
| 表单字段 | Input, Select, Switch, etc. | `base/Input` 等 |
| 底部操作栏 | Button | `base/Button` |
| 辅助面板 | Card + 说明文字 | `base/Card` |
| Schema 表单 | SchemaFormBuilder | `blocks/SchemaFormBuilder` |

### 5.3 数据流

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│  Route ID   │────▶│  API Call   │────▶│  Form State │
│  (新建/编辑) │     │  (数据获取)  │     │  (初始数据)  │
└─────────────┘     └─────────────┘     └──────┬──────┘
                                               │
                                               ▼
                                        ┌─────────────┐
                                        │  User Input │
                                        │  (表单交互)  │
                                        └──────┬──────┘
                                               │
                                               ▼
                                        ┌─────────────┐
                                        │  Validation │
                                        │  (表单校验)  │
                                        └──────┬──────┘
                                               │
                         ┌─────────────────────┼─────────────────────┐
                         ▼                     ▼                     ▼
                   ┌──────────┐          ┌──────────┐          ┌──────────┐
                   │  valid   │          │ invalid  │          │submitting│
                   │  idle    │          │  error   │          │ Spinner  │
                   └──────────┘          └──────────┘          └────┬─────┘
                                                                     │
                                                                     ▼
                                                              ┌─────────────┐
                                                              │  API Submit │
                                                              │  (提交数据)  │
                                                              └──────┬──────┘
                                                                     │
                                              ┌──────────────────────┼──────────────────────┐
                                              ▼                      ▼                      ▼
                                        ┌──────────┐          ┌──────────┐          ┌──────────┐
                                        │  success │          │  error   │          │  network │
                                        │ redirect │          │  Alert   │          │  retry   │
                                        └──────────┘          └──────────┘          └──────────┘
```

### 5.4 状态展示

#### 加载态
- 全区域居中 Spinner
- 提示文字："加载中..."
- 编辑模式：加载已有数据

#### 提交中态
- 保存按钮显示 loading
- 表单字段禁用
- 可选：显示进度条

#### 错误态
- Alert 组件，variant="error"
- 字段级错误：Input 下方显示错误文字
- 表单级错误：顶部 Alert 显示

### 5.5 使用示例

```tsx
import { FormPageShell } from '@xbis/components/layout';
import { SchemaFormBuilder } from '@xbis/components/blocks';

const CreateAbilityPage = () => {
  const [status, setStatus] = useState<FormPageStatus>('idle');
  const [values, setValues] = useState({});

  return (
    <FormPageShell
      title="发布新能力"
      subtitle="配置能力的基本信息、参数和定价"
      backPath="/abilities"
      breadcrumbs={[...]}
      status={status}
      onSubmit={handleSubmit}
      onCancel={() => navigate('/abilities')}
      submitLabel="发布能力"
      helperPanel={<AbilityPublishGuide />}
      helperPosition="right"
    >
      <SchemaFormBuilder
        fields={abilitySchemaFields}
        values={values}
        onChange={setValues}
      />
    </FormPageShell>
  );
};
```

---

## 6. 任务页模板（TaskPageShell）— 一级模板

### 6.1 页面结构

```
┌─────────────────────────────────────────────────────────────┐
│  PageHeader                                                 │
│  ├── 标题（任务中心/新建任务/任务监控）                       │
│  ├── 面包屑导航                                             │
│  └── 新建任务按钮（列表模式下）                               │
├─────────────────────────────────────────────────────────────┤
│  Status Bar（状态统计栏，列表模式）                           │
│  ├── 全部 (128)                                             │
│  ├── 执行中 (12) ●                                          │
│  ├── 队列中 (8)                                             │
│  ├── 待审核 (3)                                             │
│  ├── 已完成 (98)                                            │
│  └── 失败 (7)                                               │
├─────────────────────────────────────────────────────────────┤
│  Filter Bar（筛选栏）                                       │
│  ├── 搜索框                                                 │
│  ├── 时间范围选择                                           │
│  └── 执行模式筛选                                           │
├─────────────────────────────────────────────────────────────┤
│  Main Content Area                                          │
│  ├── 状态: loading → Spinner                                │
│  ├── 状态: empty → Empty "暂无任务"                          │
│  ├── 状态: error → Alert + 重试按钮                          │
│  ├── 状态: creating → Spinner "创建任务中..."                 │
│  └── 状态: idle → 多栏布局                                   │
│       ├── 任务列表区（60%）                                  │
│       │   ├── TaskItem 列表                                  │
│       │   └── 点击展开详情                                   │
│       └── 详情预览区（40%，可选）                             │
│           ├── 任务基本信息                                   │
│           ├── 执行时间线                                     │
│           └── 结果预览                                       │
└─────────────────────────────────────────────────────────────┘
```

### 6.2 模式切换

| 模式 | 路由示例 | 说明 |
|------|----------|------|
| list | `/tasks` | 任务列表 + 详情预览 |
| detail | `/tasks/:jobId` | 独立任务详情页 |
| create | `/tasks/create` | 新建任务表单 |
| monitor | `/tasks/monitor` | 任务监控大盘 |

### 6.3 组件组成

| 区域 | 使用组件 | 来源 |
|------|----------|------|
| 状态栏 | Tag | `base/Tag` |
| 筛选栏 | Search + DatePicker | `base/Search` |
| 任务列表 | TaskItem | `business/TaskItem` |
| 详情预览 | JobResultViewer | `blocks/JobResultViewer` |
| 时间线 | JobTimeline | `business/JobTimeline` |
| 监控统计 | StatCards | `blocks/StatCards` |

### 6.4 数据流

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│ Status Bar  │────▶│  API Call   │────▶│  Task List  │
│ (状态筛选)   │     │  (任务查询)  │     │  (任务数据)  │
└─────────────┘     └─────────────┘     └──────┬──────┘
                                               │
                                               ▼
                                        ┌─────────────┐
                                        │  Task Select│
                                        │  (选中任务)  │
                                        └──────┬──────┘
                                               │
                                               ▼
                                        ┌─────────────┐
                                        │Detail Preview│
                                        │  (详情预览)  │
                                        └─────────────┘
```

### 6.5 使用示例

```tsx
import { TaskPageShell } from '@xbis/components/layout';
import { TaskItem } from '@xbis/components/business';
import { JobResultViewer } from '@xbis/components/blocks';

const TaskCenterPage = () => {
  const [mode, setMode] = useState<TaskPageMode>('list');
  const [status, setStatus] = useState<TaskPageStatus>('idle');
  const [selectedTaskId, setSelectedTaskId] = useState<string | null>(null);

  return (
    <TaskPageShell
      title="任务中心"
      mode={mode}
      status={status}
      statusStats={[
        { status: 'running', count: 12, label: '执行中' },
        { status: 'queued', count: 8, label: '队列中' },
        { status: 'waiting_review', count: 3, label: '待审核' },
        { status: 'completed', count: 98, label: '已完成' },
        { status: 'failed', count: 7, label: '失败' },
      ]}
      selectedStatus={selectedStatus}
      onStatusChange={setSelectedStatus}
      filterBar={<TaskFilterBar onSearch={handleSearch} />}
      onCreateTask={() => setMode('create')}
      detailPreview={selectedTaskId ? <JobResultViewer job={selectedTask} /> : null}
    >
      {tasks.map(task => (
        <TaskItem
          key={task.jobId}
          task={task}
          onClick={() => setSelectedTaskId(task.jobId)}
        />
      ))}
    </TaskPageShell>
  );
};
```

---

## 7. 能力页模板（AbilityPageShell）— 一级模板

### 7.1 页面结构

```
┌─────────────────────────────────────────────────────────────┐
│  PageHeader                                                 │
│  ├── 标题（能力中心/能力详情/能力测试/接入能力）               │
│  ├── 面包屑导航                                             │
│  └── 操作按钮（测试/接入，详情模式下）                         │
├─────────────────────────────────────────────────────────────┤
│  Main Content Area                                          │
│  ├── 状态: loading → Spinner                                │
│  ├── 状态: empty → Empty "暂无可用的能力"                     │
│  ├── 状态: error → Alert + 重试按钮                          │
│  └── 状态: idle → 多栏布局                                   │
│       ├── 分类侧边栏（20%）                                  │
│       │   ├── 搜索框                                         │
│       │   ├── 分类列表                                       │
│       │   │   ├── 全部 (128)                                 │
│       │   │   ├── 数据处理 (32)                              │
│       │   │   ├── AI 模型 (24)                               │
│       │   │   └── ...                                        │
│       │   └── 标签云                                         │
│       ├── 能力网格区（50%）                                  │
│       │   └── AbilityCard 网格                               │
│       └── 详情预览区（30%，可选）                             │
│           ├── 能力基本信息                                   │
│           ├── 参数说明                                       │
│           └── 接入按钮                                       │
└─────────────────────────────────────────────────────────────┘
```

### 7.2 模式切换

| 模式 | 路由示例 | 说明 |
|------|----------|------|
| grid | `/abilities` | 能力网格 + 分类侧边栏 + 详情预览 |
| detail | `/abilities/:apiId` | 独立能力详情页 |
| test | `/abilities/:apiId/test` | 能力在线测试 |
| subscribe | `/abilities/:apiId/subscribe` | 能力接入流程 |

### 7.3 组件组成

| 区域 | 使用组件 | 来源 |
|------|----------|------|
| 分类侧边栏 | Card + Search + Tag | `base/Card`, `base/Search`, `base/Tag` |
| 能力网格 | AbilityGrid | `blocks/AbilityGrid` |
| 能力卡片 | AbilityCard | `business/AbilityCard` |
| 详情预览 | AbilityDetailPanel | `blocks/AbilityDetailPanel` |
| 测试表单 | SchemaFormBuilder | `blocks/SchemaFormBuilder` |

### 7.4 数据流

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│ Category    │────▶│  API Call   │────▶│ Ability Grid│
│ Sidebar     │     │  (能力查询)  │     │  (能力数据)  │
│ (分类筛选)   │     └─────────────┘     └──────┬──────┘
└─────────────┘                                │
                                               ▼
                                        ┌─────────────┐
                                        │ Ability Select│
                                        │  (选中能力)  │
                                        └──────┬──────┘
                                               │
                                               ▼
                                        ┌─────────────┐
                                        │Detail Preview│
                                        │  (详情预览)  │
                                        └─────────────┘
```

### 7.5 使用示例

```tsx
import { AbilityPageShell } from '@xbis/components/layout';
import { AbilityGrid } from '@xbis/components/blocks';

const AbilityCenterPage = () => {
  const [mode, setMode] = useState<AbilityPageMode>('grid');
  const [status, setStatus] = useState<AbilityPageStatus>('idle');
  const [selectedAbilityId, setSelectedAbilityId] = useState<string | null>(null);

  return (
    <AbilityPageShell
      title="能力中心"
      mode={mode}
      status={status}
      categories={[
        { key: 'all', label: '全部', count: 128 },
        { key: 'data', label: '数据处理', count: 32 },
        { key: 'ai', label: 'AI 模型', count: 24 },
      ]}
      selectedCategory={selectedCategory}
      onCategoryChange={setSelectedCategory}
      onSearch={handleSearch}
      detailPreview={selectedAbilityId ? <AbilityDetailPanel ability={selectedAbility} /> : null}
      onTestAbility={() => setMode('test')}
      onSubscribeAbility={() => setMode('subscribe')}
    >
      <AbilityGrid
        abilities={abilities}
        onAbilityClick={(ability) => setSelectedAbilityId(ability.apiId)}
      />
    </AbilityPageShell>
  );
};
```

---

## 8. 移动端适配方案

### 8.1 适配策略
- 管理端：无需移动端适配
- 用户端：核心页面支持移动端，允许功能裁剪

### 8.2 各模板移动端适配

#### ListPageShell 移动端
- 筛选栏：折叠为底部抽屉或顶部折叠面板
- 数据表格：转换为卡片列表
- 批量操作：隐藏，仅支持单条操作
- 分页：改为无限滚动或简化分页

#### DetailPageShell 移动端
- 多栏布局：改为单栏堆叠
- 侧边栏：移至主内容下方或折叠为抽屉
- 标签页：改为横向滚动或下拉选择

#### FormPageShell 移动端
- 多栏表单：改为单栏堆叠
- 辅助面板：折叠为底部 sheet 或帮助按钮
- 底部操作栏：固定悬浮于底部

#### TaskPageShell 移动端
- 状态栏：横向滚动
- 任务列表：全宽卡片
- 详情预览：点击后全屏展开（抽屉或新页面）

#### AbilityPageShell 移动端
- 分类侧边栏：折叠为顶部下拉或抽屉
- 能力网格：改为单列或双列
- 详情预览：点击后全屏展开

### 8.3 响应式断点

```ts
const breakpoints = {
  xs: '480px',   // 手机竖屏
  sm: '576px',   // 手机横屏
  md: '768px',   // 平板
  lg: '992px',   // 小型桌面
  xl: '1200px',  // 标准桌面（设计基准）
  xxl: '1600px', // 大型桌面
};
```

---

## 9. 状态管理最佳实践

### 9.1 页面状态机

每个页面模板使用统一的状态机：

```ts
type PageStatus = 'idle' | 'loading' | 'empty' | 'error';

interface PageState<T> {
  status: PageStatus;
  data: T | null;
  error: Error | null;
}

const usePageState = <T>(fetcher: () => Promise<T>) => {
  const [state, setState] = useState<PageState<T>>({
    status: 'loading',
    data: null,
    error: null,
  });

  const load = async () => {
    setState({ status: 'loading', data: null, error: null });
    try {
      const data = await fetcher();
      setState({
        status: data && (Array.isArray(data) ? data.length > 0 : true) ? 'idle' : 'empty',
        data,
        error: null,
      });
    } catch (error) {
      setState({ status: 'error', data: null, error: error as Error });
    }
  };

  return { ...state, load };
};
```

### 9.2 状态与模板映射

| 状态 | 用户反馈 | 模板行为 |
|------|----------|----------|
| loading | 显示加载动画 | 禁用交互，显示 Skeleton/Spinner |
| empty | 显示空状态插图 | 提供新建/刷新引导 |
| error | 显示错误提示 | 提供重试按钮，保留已有数据 |
| idle | 正常展示 | 启用全部交互 |

---

## 10. 文件清单

| 文件 | 说明 |
|------|------|
| `packages/components/layout/ListPageShell/index.tsx` | 列表页模板 |
| `packages/components/layout/DetailPageShell/index.tsx` | 详情页模板 |
| `packages/components/layout/FormPageShell/index.tsx` | 表单页模板 |
| `packages/components/layout/TaskPageShell/index.tsx` | 任务页模板（一级） |
| `packages/components/layout/AbilityPageShell/index.tsx` | 能力页模板（一级） |
| `docs/page-templates.md` | 本设计规范文档 |

---

*本文档与页面模板代码同步维护，新增模板时请同步更新此文档。*
