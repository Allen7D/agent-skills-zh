---
title: 策略性 Suspense 边界
impact: 高
impactDescription: 更快的初始绘制
tags: async, suspense, streaming, layout-shift
---

## 策略性 Suspense 边界

不要在返回 JSX 之前在异步组件中等待数据，而是使用 Suspense 边界在数据加载时更快地显示包装器 UI。

**错误示例（整个包装器被数据获取阻塞）：**

```tsx
async function Page() {
  const data = await fetchData() // 阻塞整个页面
  
  return (
    <div>
      <div>Sidebar</div>
      <div>Header</div>
      <div>
        <DataDisplay data={data} />
      </div>
      <div>Footer</div>
    </div>
  )
}
```

整个布局等待数据，即使只有中间部分需要它。

**正确示例（包装器立即显示，数据流入）：**

```tsx
function Page() {
  return (
    <div>
      <div>Sidebar</div>
      <div>Header</div>
      <div>
        <Suspense fallback={<Skeleton />}>
          <DataDisplay />
        </Suspense>
      </div>
      <div>Footer</div>
    </div>
  )
}

async function DataDisplay() {
  const data = await fetchData() // 只阻塞这个组件
  return <div>{data.content}</div>
}
```

Sidebar、Header 和 Footer 立即渲染。只有 DataDisplay 等待数据。

**替代方案（在组件间共享 promise）：**

```tsx
function Page() {
  // 立即启动获取，但不等待
  const dataPromise = fetchData()
  
  return (
    <div>
      <div>Sidebar</div>
      <div>Header</div>
      <Suspense fallback={<Skeleton />}>
        <DataDisplay dataPromise={dataPromise} />
        <DataSummary dataPromise={dataPromise} />
      </Suspense>
      <div>Footer</div>
    </div>
  )
}

function DataDisplay({ dataPromise }: { dataPromise: Promise<Data> }) {
  const data = use(dataPromise) // 解包 promise
  return <div>{data.content}</div>
}

function DataSummary({ dataPromise }: { dataPromise: Promise<Data> }) {
  const data = use(dataPromise) // 重用相同的 promise
  return <div>{data.summary}</div>
}
```

两个组件共享相同的 promise，所以只发生一次获取。布局立即渲染，同时两个组件一起等待。

**何时不使用这种模式：**

- 布局决策所需的关键数据（影响定位）
- 首屏上方的 SEO 关键内容
- 小型、快速查询，其中 suspense 开销不值得
- 当你想避免布局偏移（加载 → 内容跳转）

**权衡考虑：** 更快的初始绘制 vs 潜在的布局偏移。根据你的用户体验优先级来选择。
