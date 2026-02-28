# React 最佳实践

**版本 1.0.0**  
Vercel 工程团队  
2026年1月

> **注意：**  
> 本文档主要供智能体和大语言模型在维护、生成或重构 React 和 Next.js 代码库时遵循。  
> 人类开发者也可参考，但本指南针对 AI 自动化流程和一致性进行了优化。

---

## 摘要

React 和 Next.js 应用的综合性能优化指南，专为 AI 智能体和大语言模型设计。涵盖 8 个类别下 40+ 条规则，按影响力从关键（消除瀑布链、减少包体积）到增量（高级模式）排序。每条规则包含详细解释、真实案例（错误与正确实现对比）、具体影响指标，指导自动化重构和代码生成。

---

## 目录

1. [消除瀑布链](#1-eliminating-waterfalls) — **致命**
   - 1.1 [仅在需要时再 await](#11-defer-await-until-needed)
   - 1.2 [依赖驱动并行化](#12-dependency-based-parallelization)
   - 1.3 [API 路由防止瀑布链](#13-prevent-waterfall-chains-in-api-routes)
   - 1.4 [独立操作用 Promise.all()](#14-promiseall-for-independent-operations)
   - 1.5 [战略性 Suspense 边界](#15-strategic-suspense-boundaries)
2. [包体积优化](#2-bundle-size-optimization) — **致命**
   - 2.1 [避免 barrel 文件导入](#21-avoid-barrel-file-imports)
   - 2.2 [条件模块加载](#22-conditional-module-loading)
   - 2.3 [延后加载非关键三方库](#23-defer-non-critical-third-party-libraries)
   - 2.4 [重组件用动态导入](#24-dynamic-imports-for-heavy-components)
   - 2.5 [基于用户意图预加载](#25-preload-based-on-user-intent)
3. [服务端性能](#3-server-side-performance) — **高**
   - 3.1 [服务端操作需鉴权](#31-authenticate-server-actions-like-api-routes)
   - 3.2 [RSC props 避免重复序列化](#32-avoid-duplicate-serialization-in-rsc-props)
   - 3.3 [跨请求 LRU 缓存](#33-cross-request-lru-caching)
   - 3.4 [RSC 边界最小化序列化](#34-minimize-serialization-at-rsc-boundaries)
   - 3.5 [组件组合并行数据获取](#35-parallel-data-fetching-with-component-composition)
   - 3.6 [React.cache() 单请求去重](#36-per-request-deduplication-with-reactcache)
   - 3.7 [after() 非阻塞操作](#37-use-after-for-non-blocking-operations)
4. [客户端数据获取](#4-client-side-data-fetching) — **中高**
   - 4.1 [全局事件监听去重](#41-deduplicate-global-event-listeners)
   - 4.2 [滚动性能用 passive 监听](#42-use-passive-event-listeners-for-scrolling-performance)
   - 4.3 [SWR 自动去重](#43-use-swr-for-automatic-deduplication)
   - 4.4 [localStorage 数据版本化与最小化](#44-version-and-minimize-localstorage-data)
5. [重渲染优化](#5-re-render-optimization) — **中**
   - 5.1 [渲染时派生状态](#51-calculate-derived-state-during-rendering)
   - 5.2 [延迟读取状态到使用点](#52-defer-state-reads-to-usage-point)
   - 5.3 [简单原始表达式不需 useMemo](#53-do-not-wrap-a-simple-expression-with-a-primitive-result-type-in-usememo)
   - 5.4 [memo 组件默认非原始参数提为常量](#54-extract-default-non-primitive-parameter-value-from-memoized-component-to-constant)
   - 5.5 [提取为 memo 组件](#55-extract-to-memoized-components)
   - 5.6 [缩小 effect 依赖范围](#56-narrow-effect-dependencies)
   - 5.7 [交互逻辑放事件处理器](#57-put-interaction-logic-in-event-handlers)
   - 5.8 [订阅派生状态](#58-subscribe-to-derived-state)
   - 5.9 [setState 用函数式更新](#59-use-functional-setstate-updates)
   - 5.10 [useState 用懒初始化](#510-use-lazy-state-initialization)
   - 5.11 [非紧急更新用 transition](#511-use-transitions-for-non-urgent-updates)
   - 5.12 [频繁临时值用 useRef](#512-use-useref-for-transient-values)
6. [渲染性能](#6-rendering-performance) — **中**
   - 6.1 [动画应包裹 div 而非 SVG 元素](#61-animate-svg-wrapper-instead-of-svg-element)
   - 6.2 [长列表用 content-visibility](#62-css-content-visibility-for-long-lists)
   - 6.3 [静态 JSX 元素上提](#63-hoist-static-jsx-elements)
   - 6.4 [优化 SVG 精度](#64-optimize-svg-precision)
   - 6.5 [水合无闪烁防止不匹配](#65-prevent-hydration-mismatch-without-flickering)
   - 6.6 [抑制预期水合警告](#66-suppress-expected-hydration-mismatches)
   - 6.7 [显隐用 Activity 组件](#67-use-activity-component-for-showhide)
   - 6.8 [条件渲染用三元表达式](#68-use-explicit-conditional-rendering)
   - 6.9 [loading 状态优先用 useTransition](#69-use-usetransition-over-manual-loading-states)
7. [JavaScript 性能](#7-javascript-性能) — **中低**
   - 7.1 [避免布局抖动](#71-avoid-layout-thrashing)
   - 7.2 [重复查找用索引 Map](#72-build-index-maps-for-repeated-lookups)
   - 7.3 [循环中缓存属性访问](#73-cache-property-access-in-loops)
   - 7.4 [重复函数调用用缓存](#74-cache-repeated-function-calls)
   - 7.5 [缓存 Storage API 调用](#75-cache-storage-api-calls)
   - 7.6 [合并多次数组遍历](#76-combine-multiple-array-iterations)
   - 7.7 [数组比较先检查长度](#77-early-length-check-for-array-comparisons)
   - 7.8 [函数尽早 return](#78-early-return-from-functions)
   - 7.9 [RegExp 创建上提](#79-hoist-regexp-creation)
   - 7.10 [求 min/max 用循环而非 sort](#710-use-loop-for-minmax-instead-of-sort)
   - 7.11 [查找用 Set/Map 实现 O(1)](#711-use-setmap-for-o1-lookups)
   - 7.12 [不可变排序用 toSorted()](#712-use-tosorted-instead-of-sort-for-immutability)
8. [高级模式](#8-advanced-patterns) — **低**
   - 8.1 [应用仅初始化一次](#81-initialize-app-once-not-per-mount)
   - 8.2 [事件处理器存到 ref](#82-store-event-handlers-in-refs)
   - 8.3 [useEffectEvent 保持回调稳定](#83-useeffectevent-for-stable-callback-refs)

---

## 1. 消除瀑布链

**影响：致命**

瀑布链是性能杀手。每个顺序的 await 都会带来完整的网络延迟。消除它们能带来最大的收益。

### 1.1 仅在需要时再 await

**影响：高 (避免阻塞未使用的代码路径)**

将 `await` 操作移到实际使用它们的分支中，避免阻塞不需要它们的代码路径。

**错误：阻塞了两个分支**

```typescript
async function handleRequest(userId: string, skipProcessing: boolean) {
  const userData = await fetchUserData(userId)
  
  if (skipProcessing) {
    // 返回立即但仍然等待 userData
    return { skipped: true }
  }
  
  // 只有这个分支使用 userData
  return processUserData(userData)
}
```

**正确：仅在需要时阻塞**

```typescript
async function handleRequest(userId: string, skipProcessing: boolean) {
  if (skipProcessing) {
    // 返回立即而不等待
    return { skipped: true }
  }
  
  // 只在需要时获取
  const userData = await fetchUserData(userId)
  return processUserData(userData)
}
```

**另一个例子：早期返回优化**

```typescript
// 错误：总是获取权限
async function updateResource(resourceId: string, userId: string) {
  const permissions = await fetchPermissions(userId)
  const resource = await getResource(resourceId)
  
  if (!resource) {
    return { error: 'Not found' }
  }
  
  if (!permissions.canEdit) {
    return { error: 'Forbidden' }
  }
  
  return await updateResourceData(resource, permissions)
}

// 正确：仅在需要时获取
async function updateResource(resourceId: string, userId: string) {
  const resource = await getResource(resourceId)
  
  if (!resource) {
    return { error: 'Not found' }
  }
  
  const permissions = await fetchPermissions(userId)
  
  if (!permissions.canEdit) {
    return { error: 'Forbidden' }
  }
  
  return await updateResourceData(resource, permissions)
}
```

此优化尤其适用于频繁跳过分支或延迟操作昂贵的情况。

### 1.2 依赖驱动并行化

**影响：致命 (2-10× 改进)**

对于具有部分依赖的操作，使用 `better-all` 来最大化并行性。它会自动在最早可能的时刻启动每个任务。

**错误：profile 等待配置不必要的**

```typescript
const [user, config] = await Promise.all([
  fetchUser(),
  fetchConfig()
])
const profile = await fetchProfile(user.id)
```

**正确：config 和 profile 并行运行**

```typescript
import { all } from 'better-all'

const { user, config, profile } = await all({
  async user() { return fetchUser() },
  async config() { return fetchConfig() },
  async profile() {
    return fetchProfile((await this.$.user).id)
  }
})
```

**没有额外依赖的替代方案：**

```typescript
const userPromise = fetchUser()
const profilePromise = userPromise.then(user => fetchProfile(user.id))

const [user, config, profile] = await Promise.all([
  userPromise,
  fetchConfig(),
  profilePromise
])
```

我们也可以先创建所有承诺，然后在最后做 `Promise.all()`。

参考：[https://github.com/shuding/better-all](https://github.com/shuding/better-all)

### 1.3 API 路由防止瀑布链

**影响：致命 (2-10× 改进)**

在 API 路由和服务器操作中，立即启动独立操作，即使你还没有 await 它们。

**错误：config 等待 auth，data 等待两者**

```typescript
export async function GET(request: Request) {
  const session = await auth()
  const config = await fetchConfig()
  const data = await fetchData(session.user.id)
  return Response.json({ data, config })
}
```

**正确：auth 和 config 立即启动**

```typescript
export async function GET(request: Request) {
  const sessionPromise = auth()
  const configPromise = fetchConfig()
  const session = await sessionPromise
  const [config, data] = await Promise.all([
    configPromise,
    fetchData(session.user.id)
  ])
  return Response.json({ data, config })
}
```

对于具有更复杂依赖链的操作，使用 `better-all` 来自动最大化并行性（见依赖驱动并行化）。

### 1.4 独立操作用 Promise.all()

**影响：致命 (2-10× 改进)**

当异步操作没有相互依赖时，使用 `Promise.all()` 并行执行。

**错误：顺序执行，3 次往返**

```typescript
const user = await fetchUser()
const posts = await fetchPosts()
const comments = await fetchComments()
```

**正确：并行执行，1 次往返**

```typescript
const [user, posts, comments] = await Promise.all([
  fetchUser(),
  fetchPosts(),
  fetchComments()
])
```

### 1.5 战略性 Suspense 边界

**影响：高 (更快的初始渲染)**

在异步组件返回 JSX 之前，使用 Suspense 边界来显示包装 UI，而数据加载。

**错误：包装被数据获取阻塞**

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

**正确：包装立即显示，数据流进来**

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
  const data = await fetchData() // 只有这个组件被阻塞
  return <div>{data.content}</div>
}
```

Sidebar、Header、Footer 立即渲染。只有 DataDisplay 等待数据。

**替代方案：共享 promise**

```tsx
function Page() {
  // 立即开始获取，但不 await
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
  const data = use(dataPromise) // 重用同一个 promise
  return <div>{data.summary}</div>
}
```

两个组件共享同一个 promise，所以只发生一次 fetch。布局立即渲染，而两个组件一起等待。

**何时不要使用此模式：**

- 关键数据需要布局决策（影响定位）

- SEO-critical 内容在折叠之上

- 小、快的查询，其中 suspense 开销不值得

- 当你想要避免布局移位（加载 → 内容跳转）

**权衡：** 更快的初始渲染 vs 可能的布局移位。根据你的 UX 优先级选择。

---

## 2. 包体积优化

**影响：致命**

减少初始包体积能提高 Time to Interactive 和 Largest Contentful Paint。

### 2.1 避免 barrel 文件导入

**影响：致命 (200-800ms 导入成本，慢构建)**

直接从源文件导入，而不是 barrel 文件，以避免加载数千个未使用的模块。**Barrel 文件**是入口点，重新导出多个模块（例如，`index.js` 通过 `export * from './module'`）。

流行的图标和组件库可以有**多达 10,000 个 re-exports**在它们的入口文件中。对于许多 React 包，**它需要 200-800ms 仅导入它们**，影响开发速度和生产冷启动。

**为什么 tree-shaking 不帮助：**当一个库被标记为外部（不捆绑）时，捆绑器不能优化它。如果你捆绑它以启用 tree-shaking，构建会变得显著慢下来分析整个模块图。

**错误：导入整个库**

```tsx
import { Check, X, Menu } from 'lucide-react'
// 加载 1,583 模块，需要 ~2.8s 在开发中
// 运行时成本：200-800ms 在每次冷启动

import { Button, TextField } from '@mui/material'
// 加载 2,225 模块，需要 ~4.2s 在开发中
```

**正确：只导入你需要的**

```tsx
import Check from 'lucide-react/dist/esm/icons/check'
import X from 'lucide-react/dist/esm/icons/x'
import Menu from 'lucide-react/dist/esm/icons/menu'
// 只加载 3 模块 (~2KB vs ~1MB)

import Button from '@mui/material/Button'
import TextField from '@mui/material/TextField'
// 只加载你使用的东西
```

**替代方案：Next.js 13.5+**

```js
// next.config.js - use optimizePackageImports
module.exports = {
  experimental: {
    optimizePackageImports: ['lucide-react', '@mui/material']
  }
}

// 然后你可以保持简洁的 barrel 导入：
import { Check, X, Menu } from 'lucide-react'
// 自动转换为直接导入在构建时
```

直接导入提供 15-70% 更快的开发启动，28% 更快的构建，40% 更快的冷启动，以及显著更快的 HMR。

受影响的库：`lucide-react`, `@mui/material`, `@mui/icons-material`, `@tabler/icons-react`, `react-icons`, `@headlessui/react`, `@radix-ui/react-*`, `lodash`, `ramda`, `date-fns`, `rxjs`, `react-use`。

参考：[https://vercel.com/blog/how-we-optimized-package-imports-in-next-js](https://vercel.com/blog/how-we-optimized-package-imports-in-next-js)

### 2.2 条件模块加载

**影响：高 (仅在需要时加载大型数据)**

仅在特征激活时加载大型数据或模块。

**例子：懒加载动画帧**

```tsx
function AnimationPlayer({ enabled, setEnabled }: { enabled: boolean; setEnabled: React.Dispatch<React.SetStateAction<boolean>> }) {
  const [frames, setFrames] = useState<Frame[] | null>(null)

  useEffect(() => {
    if (enabled && !frames && typeof window !== 'undefined') {
      import('./animation-frames.js')
        .then(mod => setFrames(mod.frames))
        .catch(() => setEnabled(false))
    }
  }, [enabled, frames, setEnabled])

  if (!frames) return <Skeleton />
  return <Canvas frames={frames} />
}
```

`typeof window !== 'undefined'` 检查防止在 SSR 中捆绑此模块，优化服务器包大小和构建速度。

### 2.3 延后加载非关键三方库

**影响：中 (在水合后加载)**

分析、日志、错误跟踪不会阻塞用户交互。在水合后加载。

**错误：阻塞初始包**

```tsx
import { Analytics } from '@vercel/analytics/react'

export default function RootLayout({ children }) {
  return (
    <html>
      <body>
        {children}
        <Analytics />
      </body>
    </html>
  )
}
```

**正确：在水合后加载**

```tsx
import dynamic from 'next/dynamic'

const Analytics = dynamic(
  () => import('@vercel/analytics/react').then(m => m.Analytics),
  { ssr: false }
)

export default function RootLayout({ children }) {
  return (
    <html>
      <body>
        {children}
        <Analytics />
      </body>
    </html>
  )
}
```

### 2.4 重组件用动态导入

**影响：致命 (直接影响 TTI 和 LCP)**

使用 `next/dynamic` 来懒加载大型组件，不需要在初始渲染时加载。

**错误：Monaco 与主包 ~300KB**

```tsx
import { MonacoEditor } from './monaco-editor'

function CodePanel({ code }: { code: string }) {
  return <MonacoEditor value={code} />
}
```

**正确：Monaco 按需加载**

```tsx
import dynamic from 'next/dynamic'

const MonacoEditor = dynamic(
  () => import('./monaco-editor').then(m => m.MonacoEditor),
  { ssr: false }
)

function CodePanel({ code }: { code: string }) {
  return <MonacoEditor value={code} />
}
```

### 2.5 基于用户意图预加载

**影响：中 (减少感知延迟)**

在需要之前预加载重包，以减少感知延迟。

**例子：在 hover/focus 时预加载**

```tsx
function EditorButton({ onClick }: { onClick: () => void }) {
  const preload = () => {
    if (typeof window !== 'undefined') {
      void import('./monaco-editor')
    }
  }

  return (
    <button
      onMouseEnter={preload}
      onFocus={preload}
      onClick={onClick}
    >
      Open Editor
    </button>
  )
}
```

**例子：当特征标志启用时预加载**

```tsx
function FlagsProvider({ children, flags }: Props) {
  useEffect(() => {
    if (flags.editorEnabled && typeof window !== 'undefined') {
      void import('./monaco-editor').then(mod => mod.init())
    }
  }, [flags.editorEnabled])

  return <FlagsContext.Provider value={flags}>
    {children}
  </FlagsContext.Provider>
}
```

`typeof window !== 'undefined'` 检查防止在 SSR 中捆绑预加载模块，优化服务器包大小和构建速度。

---

## 3. 服务端性能

**影响：高**

优化服务端渲染和数据获取消除了服务端瀑布链并减少了响应时间。

### 3.1 服务端操作需鉴权

**影响：致命 (防止未经授权访问服务器突变)**

服务端操作（带有 `"use server"` 的函数）是公开端点，就像 API 路由一样。总是验证认证和授权 **在每个服务端操作内部**—不要仅依赖于中间件、布局守卫或页面级检查，因为服务端操作可以被直接调用。

Next.js 文档明确指出：“将服务端操作与公开端点的相同安全考虑，验证用户是否允许执行突变。”

**错误：没有认证检查**

```typescript
'use server'

export async function deleteUser(userId: string) {
  // 任何人都可以调用这个！没有认证检查
  await db.user.delete({ where: { id: userId } })
  return { success: true }
}
```

**正确：在操作内部认证**

```typescript
'use server'

import { verifySession } from '@/lib/auth'
import { unauthorized } from '@/lib/errors'

export async function deleteUser(userId: string) {
  // 总是检查认证
  const session = await verifySession()
  
  if (!session) {
    throw unauthorized('Must be logged in')
  }
  
  // 检查授权
  if (session.user.role !== 'admin' && session.user.id !== userId) {
    throw unauthorized('Cannot delete other users')
  }
  
  await db.user.delete({ where: { id: userId } })
  return { success: true }
}
```

**带输入验证：**

```typescript
'use server'

import { verifySession } from '@/lib/auth'
import { z } from 'zod'

const updateProfileSchema = z.object({
  userId: z.string().uuid(),
  name: z.string().min(1).max(100),
  email: z.string().email()
})

export async function updateProfile(data: unknown) {
  // 验证输入
  const validated = updateProfileSchema.parse(data)
  
  // 然后认证
  const session = await verifySession()
  if (!session) {
    throw new Error('Unauthorized')
  }
  
  // 然后授权
  if (session.user.id !== validated.userId) {
    throw new Error('Can only update own profile')
  }
  
  // 最后执行突变
  await db.user.update({
    where: { id: validated.userId },
    data: {
      name: validated.name,
      email: validated.email
    }
  })
  
  return { success: true }
}
```

参考：[https://nextjs.org/docs/app/guides/authentication](https://nextjs.org/docs/app/guides/authentication)

### 3.2 避免 RSC props 中的重复序列化

**影响：低 (减少网络负载)**

RSC→client 序列化通过对象引用而不是值。相同引用 = 一次序列化；新引用 = 再次序列化。在客户端进行变换（`.toSorted()`, `.filter()`, `.map()`)，而不是在服务器端。

**错误：重复数组**

```tsx
// RSC: 发送 6 个字符串 (2 个数组 × 3 个项目)
<ClientList usernames={usernames} usernamesOrdered={usernames.toSorted()} />
```

**正确：发送 3 个字符串**

```tsx
// RSC: 发送一次
<ClientList usernames={usernames} />

// 客户端：变换那里
'use client'
const sorted = useMemo(() => [...usernames].sort(), [usernames])
```

**嵌套去重行为：**

```tsx
// string[] - 复制一切
usernames={['a','b']} sorted={usernames.toSorted()} // 发送 4 个字符串

// object[] - 复制数组结构
users={[{id:1},{id:2}]} sorted={users.toSorted()} // 发送 2 个数组 + 2 个唯一对象 (不是 4 个)
```

去重是递归的。影响因数据类型而异：

- `string[]`, `number[]`, `boolean[]`: **高影响** - 数组 + 所有原始值完全复制

- `object[]`: **低影响** - 数组被复制，但嵌套对象通过引用去重

**破坏去重的行为：创建新引用**

- 数组：`.toSorted()`, `.filter()`, `.map()`, `.slice()`, `[...arr]`

- 对象：`{...obj}`, `Object.assign()`, `structuredClone()`, `JSON.parse(JSON.stringify())`

**更多例子：**

```tsx
// ❌ Bad
<C users={users} active={users.filter(u => u.active)} />
<C product={product} productName={product.name} />

// ✅ Good
<C users={users} />
<C product={product} />
// 在客户端进行过滤/解构
```

**例外：** 当变换是昂贵或客户端不需要原始数据时，传递派生数据。

### 3.3 跨请求 LRU 缓存

**影响：高 (跨请求缓存)**

`React.cache()` 只在单个请求内工作。对于在连续请求中共享数据的场景（用户点击按钮 A 然后点击按钮 B），使用 LRU 缓存。

**实现：**

```typescript
import { LRUCache } from 'lru-cache'

const cache = new LRUCache<string, any>({
  max: 1000,
  ttl: 5 * 60 * 1000  // 5 分钟
})

export async function getUser(id: string) {
  const cached = cache.get(id)
  if (cached) return cached

  const user = await db.user.findUnique({ where: { id } })
  cache.set(id, user)
  return user
}

// 请求 1: DB 查询，结果缓存
// 请求 2: 缓存命中，无 DB 查询
```

使用场景：用户连续点击多个端点需要相同数据。

**与 Vercel 的 [Fluid Compute](https://vercel.com/docs/fluid-compute):** LRU 缓存尤其有效，因为多个并发请求可以共享相同的函数实例和缓存。这意味着缓存持久化，无需外部存储如 Redis。

**在传统无服务器中：** 每次调用都在隔离中运行，所以考虑 Redis 用于跨进程缓存。

参考：[https://github.com/isaacs/node-lru-cache](https://github.com/isaacs/node-lru-cache)

### 3.4 最小化 RSC 边界序列化

**影响：高 (减少数据传输大小)**

React Server/Client 边界将所有对象属性序列化为字符串并嵌入 HTML 响应和后续 RSC 请求中。这直接影晌页面重量和加载时间，所以 **大小很重要**。只传递客户端实际使用的字段。

**错误：序列化所有 50 个字段**

```tsx
async function Page() {
  const user = await fetchUser()  // 50 个字段
  return <Profile user={user} />
}

'use client'
function Profile({ user }: { user: User }) {
  return <div>{user.name}</div>  // 使用 1 个字段
}
```

**正确：只序列化 1 个字段**

```tsx
async function Page() {
  const user = await fetchUser()
  return <Profile name={user.name} />
}

'use client'
function Profile({ name }: { name: string }) {
  return <div>{name}</div>
}
```

### 3.5 并行数据获取与组件组合

**影响：致命 (消除服务端瀑布链)**

React Server Components 在树内执行顺序。重构为组合以并行化数据获取。

**错误：Sidebar 等待 Page 的 fetch 完成**

```tsx
export default async function Page() {
  const header = await fetchHeader()
  return (
    <div>
      <div>{header}</div>
      <Sidebar />
    </div>
  )
}

async function Sidebar() {
  const items = await fetchSidebarItems()
  return <nav>{items.map(renderItem)}</nav>
}
```

**正确：同时获取**

```tsx
async function Header() {
  const data = await fetchHeader()
  return <div>{data}</div>
}

async function Sidebar() {
  const items = await fetchSidebarItems()
  return <nav>{items.map(renderItem)}</nav>
}

export default function Page() {
  return (
    <div>
      <Header />
      <Sidebar />
    </div>
  )
}
```

**带 children prop 的替代方案：**

```tsx
async function Header() {
  const data = await fetchHeader()
  return <div>{data}</div>
}

async function Sidebar() {
  const items = await fetchSidebarItems()
  return <nav>{items.map(renderItem)}</nav>
}

function Layout({ children }: { children: ReactNode }) {
  return (
    <div>
      <Header />
      {children}
    </div>
  )
}

export default function Page() {
  return (
    <Layout>
      <Sidebar />
    </Layout>
  )
}
```

### 3.6 React.cache() 单请求去重

**影响：中 (请求内去重)**

使用 `React.cache()` 进行服务端请求去重。认证和数据库查询受益最多。

**用法：**

```typescript
import { cache } from 'react'

export const getCurrentUser = cache(async () => {
  const session = await auth()
  if (!session?.user?.id) return null
  return await db.user.findUnique({
    where: { id: session.user.id }
  })
})
```

在单个请求内，多次调用 `getCurrentUser()` 执行查询仅一次。

**避免 inline objects 作为参数：**

`React.cache()` 使用浅等性 (`Object.is`) 来确定缓存命中。inline objects 创建新引用每次调用，防止缓存命中。

**错误：总是缓存未命中**

```typescript
const getUser = cache(async (params: { uid: number }) => {
  return await db.user.findUnique({ where: { id: params.uid } })
})

// 每次调用创建新对象，从未命中缓存
getUser({ uid: 1 })
getUser({ uid: 1 })  // 缓存未命中，再次运行查询
```

**正确：缓存命中**

```typescript
const params = { uid: 1 }
getUser(params)  // 查询运行
getUser(params)  // 缓存命中 (相同引用)
```

如果必须传递对象，传递相同的引用：

**Next.js-Specific Note:**

在 Next.js 中，`fetch` API 自动扩展为请求记忆化。具有相同 URL 和选项的请求在单个请求内自动去重，所以你不需要 `React.cache()` 对 `fetch` 调用。然而，`React.cache()` 仍然对其他异步任务至关重要：

- 数据库查询 (Prisma, Drizzle, 等)

- 重计算

- 认证检查

- 文件系统操作

- 任何非-fetch 异步工作

使用 `React.cache()` 来去重这些操作。

参考：[https://react.dev/reference/react/cache](https://react.dev/reference/react/cache)

### 3.7 使用 after() 非阻塞操作

**影响：中 (更快的响应时间)**

使用 Next.js 的 `after()` 来调度应在响应发送后执行的工作。这防止了日志、分析和其他副作用阻塞响应。

**错误：阻塞响应**

```tsx
import { logUserAction } from '@/app/utils'

export async function POST(request: Request) {
  // 执行突变
  await updateDatabase(request)
  
  // 日志阻塞响应
  const userAgent = request.headers.get('user-agent') || 'unknown'
  await logUserAction({ userAgent })
  
  return new Response(JSON.stringify({ status: 'success' }), {
    status: 200,
    headers: { 'Content-Type': 'application/json' }
  })
}
```

**正确：非阻塞**

```tsx
import { after } from 'next/server'
import { headers, cookies } from 'next/headers'
import { logUserAction } from '@/app/utils'

export async function POST(request: Request) {
  // 执行突变
  await updateDatabase(request)
  
  // 日志在响应发送后
  after(async () => {
    const userAgent = (await headers()).get('user-agent') || 'unknown'
    const sessionCookie = (await cookies()).get('session-id')?.value || 'anonymous'
    
    logUserAction({ sessionCookie, userAgent })
  })
  
  return new Response(JSON.stringify({ status: 'success' }), {
    status: 200,
    headers: { 'Content-Type': 'application/json' }
  })
}
```

响应立即发送，而日志在后台发生。

**常见用例：**

- 分析跟踪

- 审计日志

- 发送通知

- 缓存无效化

- 清理任务

**重要笔记：**

- `after()` 即使响应失败或重定向也会运行

- 在 Server Actions、Route Handlers、Server Components 中运行

参考：[https://nextjs.org/docs/app/api-reference/functions/after](https://nextjs.org/docs/app/api-reference/functions/after)

---

## 4. 客户端数据获取

**影响：中高**

自动去重和高效的获取模式减少了冗余网络请求。

### 4.1 去重全局事件监听

**影响：低 (单监听器为 N 组件)**

使用 `useSWRSubscription()` 在组件实例之间共享全局事件监听器。

**错误：N 实例 = N 监听器**

```tsx
function useKeyboardShortcut(key: string, callback: () => void) {
  useEffect(() => {
    const handler = (e: KeyboardEvent) => {
      if (e.metaKey && e.key === key) {
        callback()
      }
    }
    window.addEventListener('keydown', handler)
    return () => window.removeEventListener('keydown', handler)
  }, [key, callback])
}
```

当使用 `useKeyboardShortcut` 多次时，每个实例会注册一个新监听器。

**正确：N 实例 = 1 监听器**

```tsx
import useSWRSubscription from 'swr/subscription'

// 模块级 Map 跟踪每个键的回调
const keyCallbacks = new Map<string, Set<() => void>>()

function useKeyboardShortcut(key: string, callback: () => void) {
  // 注册这个回调在 Map 中
  useEffect(() => {
    if (!keyCallbacks.has(key)) {
      keyCallbacks.set(key, new Set())
    }
    keyCallbacks.get(key)!.add(callback)

    return () => {
      const set = keyCallbacks.get(key)
      if (set) {
        set.delete(callback)
        if (set.size === 0) {
          keyCallbacks.delete(key)
        }
      }
    }
  }, [key, callback])

  useSWRSubscription('global-keydown', () => {
    const handler = (e: KeyboardEvent) => {
      if (e.metaKey && keyCallbacks.has(e.key)) {
        keyCallbacks.get(e.key)!.forEach(cb => cb())
      }
    }
    window.addEventListener('keydown', handler)
    return () => window.removeEventListener('keydown', handler)
  })
}

function Profile() {
  // 多个快捷键共享同一个监听器
  useKeyboardShortcut('p', () => { /* ... */ }) 
  useKeyboardShortcut('k', () => { /* ... */ })
  // ...
}
```

### 4.2 使用 passive 监听器用于滚动性能

**影响：中 (消除滚动延迟造成的事件监听器)**

添加 `{ passive: true }` 到触摸和轮盘事件监听器，以启用立即滚动。浏览器通常会等待监听器完成以检查 `preventDefault()` 是否被调用，导致滚动延迟。

**错误：**

```typescript
useEffect(() => {
  const handleTouch = (e: TouchEvent) => console.log(e.touches[0].clientX)
  const handleWheel = (e: WheelEvent) => console.log(e.deltaY)
  
  document.addEventListener('touchstart', handleTouch)
  document.addEventListener('wheel', handleWheel)
  
  return () => {
    document.removeEventListener('touchstart', handleTouch)
    document.removeEventListener('wheel', handleWheel)
  }
}, [])
```

**正确：**

```typescript
useEffect(() => {
  const handleTouch = (e: TouchEvent) => console.log(e.touches[0].clientX)
  const handleWheel = (e: WheelEvent) => console.log(e.deltaY)
  
  document.addEventListener('touchstart', handleTouch, { passive: true })
  document.addEventListener('wheel', handleWheel, { passive: true })
  
  return () => {
    document.removeEventListener('touchstart', handleTouch)
    document.removeEventListener('wheel', handleWheel)
  }
}, [])
```

**使用 passive 当：** 跟踪/分析、日志、任何不调用 `preventDefault()` 的监听器。

**不要使用 passive 当：** 实现自定义的滑动手势、自定义的缩放控制、或任何需要 `preventDefault()` 的监听器。

### 4.3 使用 SWR 进行自动去重

**影响：中高 (自动去重)**

SWR 启用请求去重、缓存和重新验证。

**错误：没有去重，每个实例都获取**

```tsx
function UserList() {
  const [users, setUsers] = useState([])
  useEffect(() => {
    fetch('/api/users')
      .then(r => r.json())
      .then(setUsers)
  }, [])
}
```

**正确：多个实例共享一个请求**

```tsx
import useSWR from 'swr'

function UserList() {
  const { data: users } = useSWR('/api/users', fetcher)
}
```

**对于不可变数据：**

```tsx
import { useImmutableSWR } from '@/lib/swr'

function StaticContent() {
  const { data } = useImmutableSWR('/api/config', fetcher)
}
```

**对于突变：**

```tsx
import { useSWRMutation } from 'swr/mutation'

function UpdateButton() {
  const { trigger } = useSWRMutation('/api/user', updateUser)
  return <button onClick={() => trigger()}>Update</button>
}
```

参考：[https://swr.vercel.app](https://swr.vercel.app)

### 4.4 版本化和最小化 localStorage 数据

**影响：中 (防止 schema 冲突，减少存储大小)**

添加版本前缀到键名并只存储需要的字段。防止 schema 冲突和意外存储敏感数据。

**错误：**

```typescript
// 没有版本，存储了所有内容，没有错误处理
localStorage.setItem('userConfig', JSON.stringify(fullUserObject))
const data = localStorage.getItem('userConfig')
```

**正确：**

```typescript
const VERSION = 'v2'

function saveConfig(config: { theme: string; language: string }) {
  try {
    localStorage.setItem(`userConfig:${VERSION}`, JSON.stringify(config))
  } catch {
    // Throws in incognito/private browsing, quota exceeded, or disabled
  }
}

function loadConfig() {
  try {
    const data = localStorage.getItem(`userConfig:${VERSION}`)
    return data ? JSON.parse(data) : null
  } catch {
    return null
  }
}

// 迁移从 v1 到 v2
function migrate() {
  try {
    const v1 = localStorage.getItem('userConfig:v1')
    if (v1) {
      const old = JSON.parse(v1)
      saveConfig({ theme: old.darkMode ? 'dark' : 'light', language: old.lang })
      localStorage.removeItem('userConfig:v1')
    }
  } catch {}
}
```

**从服务器响应中存储最小字段：**

```typescript
// 用户对象有 20+ 字段，只存储 UI 需要的
function cachePrefs(user: FullUser) {
  try {
    localStorage.setItem('prefs:v1', JSON.stringify({
      theme: user.preferences.theme,
      notifications: user.preferences.notifications
    }))
  } catch {}
}
```

**总是用 try-catch 包裹：** `getItem()` 和 `setItem()` 在 incognito/private 浏览器中会抛出错误（Safari, Firefox），当 quota 超过或禁用时。

**好处：** Schema 演进通过版本化，减少存储大小，防止存储令牌/PII/内部标志。

---

## 5. 重渲染优化

**影响：中**

减少不必要的重渲染可以最小化浪费的计算和提高 UI 响应性。

### 5.1 渲染时派生状态

**影响：中 (避免冗余渲染和状态漂移)**

如果一个值可以从当前 props/state 计算出来，不要存储在 state 或 effect 中。派生它在渲染时，避免额外的渲染和状态漂移。不要在 effect 中仅响应 prop 变化；优先使用派生值或键重置。

**错误：冗余状态和 effect**

```tsx
function Form() {
  const [firstName, setFirstName] = useState('First')
  const [lastName, setLastName] = useState('Last')
  const [fullName, setFullName] = useState('')

  useEffect(() => {
    setFullName(firstName + ' ' + lastName)
  }, [firstName, lastName])

  return <p>{fullName}</p>
}
```

**正确：派生期间**

```tsx
function Form() {
  const [firstName, setFirstName] = useState('First')
  const [lastName, setLastName] = useState('Last')
  const fullName = firstName + ' ' + lastName

  return <p>{fullName}</p>
}
```

参考：[https://react.dev/learn/you-might-not-need-an-effect](https://react.dev/learn/you-might-not-need-an-effect)

### 5.2 延迟读取状态到使用点

**影响：中 (避免不必要的订阅)**

不要订阅动态状态（searchParams, localStorage）如果只在回调中读取。

**错误：订阅所有 searchParams 变化**

```tsx
function ShareButton({ chatId }: { chatId: string }) {
  const searchParams = useSearchParams()

  const handleShare = () => {
    const ref = searchParams.get('ref')
    shareChat(chatId, { ref })
  }

  return <button onClick={handleShare}>Share</button>
}
```

**正确：按需读取，无订阅**

```tsx
function ShareButton({ chatId }: { chatId: string }) {
  const handleShare = () => {
    const params = new URLSearchParams(window.location.search)
    const ref = params.get('ref')
    shareChat(chatId, { ref })
  }

  return <button onClick={handleShare}>Share</button>
}
```

### 5.3 不要将简单表达式用 primitive 结果类型包裹在 useMemo 中

**影响：低-中 (浪费每次渲染的计算)**

当表达式是简单的（少数逻辑或算术运算）且有 primitive 结果类型（布尔、数字、字符串）时，不要用 `useMemo` 包裹。

调用 `useMemo` 和比较 hook 依赖可能会消耗比表达式本身更多的资源。

**错误：**

```tsx
function Header({ user, notifications }: Props) {
  const isLoading = useMemo(() => {
    return user.isLoading || notifications.isLoading
  }, [user.isLoading, notifications.isLoading])

  if (isLoading) return <Skeleton />
  // return some markup
}
```

**正确：**

```tsx
function Header({ user, notifications }: Props) {
  const isLoading = user.isLoading || notifications.isLoading

  if (isLoading) return <Skeleton />
  // return some markup
}
```

### 5.4 提取默认非原始参数值为常量

**影响：中 (恢复 memoization 用常量代替默认值)**

当 memoized 组件有某些非原始可选参数的默认值，如数组、函数或对象时，不带该参数调用组件会导致 memoization 失败。这是因为每次重新渲染都会创建新实例，而它们不会通过 `memo()` 的严格相等比较。

为了解决这个问题，提取默认值为常量。

**错误：`onClick` 在每次重新渲染时值不同**

```tsx
const UserAvatar = memo(function UserAvatar({ onClick = () => {} }: { onClick?: () => void }) {
  // ...
})

// 用于没有 optional onClick
<UserAvatar />
```

**正确：稳定默认值**

```tsx
const NOOP = () => {};

const UserAvatar = memo(function UserAvatar({ onClick = NOOP }: { onClick?: () => void }) {
  // ...
})

// 用于没有 optional onClick
<UserAvatar />
```

### 5.5 提取为 memo 组件

**影响：中 (启用早期返回)**

将昂贵的工作提取为 memo 组件，以在计算前启用早期返回。

**错误：计算 avatar 即使在加载时**

```tsx
function Profile({ user, loading }: Props) {
  const avatar = useMemo(() => {
    const id = computeAvatarId(user)
    return <Avatar id={id} />
  }, [user])

  if (loading) return <Skeleton />
  return <div>{avatar}</div>
}
```

**正确：加载时跳过计算**

```tsx
const UserAvatar = memo(function UserAvatar({ user }: { user: User }) {
  const id = useMemo(() => computeAvatarId(user), [user])
  return <Avatar id={id} />
})

function Profile({ user, loading }: Props) {
  if (loading) return <Skeleton />
  return (
    <div>
      <UserAvatar user={user} />
    </div>
  )
}
```

**注意：** 如果你的项目有 [React Compiler](https://react.dev/learn/react-compiler) 启用，手动 memoization 用 `memo()` 和 `useMemo()` 不是必要的。编译器自动优化重新渲染。

### 5.6 缩小 effect 依赖范围

**影响：低 (最小化 effect 重新运行)**

指定原始依赖而不是对象，以最小化 effect 重新运行。

**错误：任何用户字段变化都重新运行**

```tsx
useEffect(() => {
  console.log(user.id)
}, [user])
```

**正确：仅当 id 变化时重新运行**

```tsx
useEffect(() => {
  console.log(user.id)
}, [user.id])
```

**对于派生状态，计算在 effect 外：**

```tsx
// 错误：在 width=767, 766, 765...
useEffect(() => {
  if (width < 768) {
    enableMobileMode()
  }
}, [width])

// 正确：仅在 boolean 转换时运行
const isMobile = width < 768
useEffect(() => {
  if (isMobile) {
    enableMobileMode()
  }
}, [isMobile])
```

### 5.7 将交互逻辑放在事件处理器中

**影响：中 (避免 effect 重新运行和重复副作用)**

如果一个副作用是由特定用户操作（提交、点击、拖拽）触发的，就在那个事件处理器中运行它。不要将动作建模为 state + effect；这会使 effect 在无关变化上重新运行并可能重复动作。

**错误：事件建模为 state + effect**

```tsx
function Form() {
  const [submitted, setSubmitted] = useState(false)
  const theme = useContext(ThemeContext)

  useEffect(() => {
    if (submitted) {
      post('/api/register')
      showToast('Registered', theme)
    }
  }, [submitted, theme])

  return <button onClick={() => setSubmitted(true)}>Submit</button>
}
```

**正确：在处理器中执行**

```tsx
function Form() {
  const theme = useContext(ThemeContext)

  function handleSubmit() {
    post('/api/register')
    showToast('Registered', theme)
  }

  return <button onClick={handleSubmit}>Submit</button>
}
```

参考：[https://react.dev/learn/removing-effect-dependencies#should-this-code-move-to-an-event-handler](https://react.dev/learn/removing-effect-dependencies#should-this-code-move-to-an-event-handler)

### 5.8 订阅派生状态

**影响：中 (减少重渲染频率)**

订阅派生布尔状态而不是连续值，以减少重渲染频率。

**错误：每像素变化都重新渲染**

```tsx
function Sidebar() {
  const width = useWindowWidth()  // 更新连续
  const isMobile = width < 768
  return <nav className={isMobile ? 'mobile' : 'desktop'} />
}
```

**正确：仅当布尔值变化时重新渲染**

```tsx
function Sidebar() {
  const isMobile = useMediaQuery('(max-width: 767px)')
  return <nav className={isMobile ? 'mobile' : 'desktop'} />
}
```

### 5.9 使用函数式 setState 更新

**影响：中 (防止 stale closures 和不必要的 callback 重建)**

当基于当前状态值更新状态时，使用 setState 的函数式更新形式，而不是直接引用状态变量。这防止了 stale closures，消除了不必要的依赖，创建了稳定的 callback 参考。

**错误：需要 state 作为依赖**

```tsx
function TodoList() {
  const [items, setItems] = useState(initialItems)
  
  // Callback 必须依赖 items，每次 items 变化都会重建
  const addItems = useCallback((newItems: Item[]) => {
    setItems([...items, ...newItems])
  }, [items])  // ❌ items 依赖导致重建
  
  // stale closure 风险
  const removeItem = useCallback((id: string) => {
    setItems(items.filter(item => item.id !== id))
  }, [])  // ❌ 缺少 items 依赖 - 会使用 stale items!
  
  return <ItemsEditor items={items} onAdd={addItems} onRemove={removeItem} />
}
```

第一个回调每次 `items` 变化都会重建，这可能导致子组件不必要的重新渲染。第二个回调有 stale closure 问题——它总是引用初始 `items` 值。

**正确：稳定回调，无 stale closures**

```tsx
function TodoList() {
  const [items, setItems] = useState(initialItems)
  
  // 稳定回调，永不重建
  const addItems = useCallback((newItems: Item[]) => {
    setItems(curr => [...curr, ...newItems])
  }, [])  // ✅ 无依赖
  
  // 总是使用最新状态，无 stale closure 风险
  const removeItem = useCallback((id: string) => {
    setItems(curr => curr.filter(item => item.id !== id))
  }, [])  // ✅ 安全且稳定
  
  return <ItemsEditor items={items} onAdd={addItems} onRemove={removeItem} />
}
```

**好处：**

1. **Stable callback references** - Callbacks don't need to be recreated when state changes

2. **No stale closures** - Always operates on the latest state value

3. **Fewer dependencies** - Simplifies dependency arrays and reduces memory leaks

4. **Prevents bugs** - Eliminates the most common source of React closure bugs

**何时使用函数式更新：**

- 任何 setState 依赖于当前状态值

- 在 useCallback/useMemo 中需要 state

- 事件处理器引用 state

- 异步操作更新 state

**何时直接更新是合适的：**

- 设置 state 为静态值：`setCount(0)`

- 设置 state 从 props/arguments 仅：`setName(newName)`

- State 不依赖于前一个值

**注意：** 如果你的项目有 [React Compiler](https://react.dev/learn/react-compiler) 启用，编译器可以自动优化一些情况，但函数式更新仍然是推荐的，以确保正确性和防止 stale closure bugs。

### 5.10 使用 Lazy State 初始化

**影响：中 (浪费每次渲染的计算)**

将函数传递给 `useState` 用于昂贵的初始值。没有函数形式，初始化器在每次渲染时都会运行，即使值只使用一次。

**错误：每次渲染都运行**

```tsx
function FilteredList({ items }: { items: Item[] }) {
  // buildSearchIndex() 在每次渲染时都运行，即使初始化后
  const [searchIndex, setSearchIndex] = useState(buildSearchIndex(items))
  const [query, setQuery] = useState('')
  
  // 当 query 变化时，buildSearchIndex 会再次运行
  return <SearchResults index={searchIndex} query={query} />
}

function UserProfile() {
  // JSON.parse 在每次渲染时都运行
  const [settings, setSettings] = useState(
    JSON.parse(localStorage.getItem('settings') || '{}')
  )
  
  return <SettingsForm settings={settings} onChange={setSettings} />
}
```

**正确：仅运行一次**

```tsx
function FilteredList({ items }: { items: Item[] }) {
  // buildSearchIndex() 仅在初始渲染时运行
  const [searchIndex, setSearchIndex] = useState(() => buildSearchIndex(items))
  const [query, setQuery] = useState('')
  
  return <SearchResults index={searchIndex} query={query} />
}

function UserProfile() {
  // JSON.parse 仅在初始渲染时运行
  const [settings, setSettings] = useState(() => {
    const stored = localStorage.getItem('settings')
    return stored ? JSON.parse(stored) : {}
  })
  
  return <SettingsForm settings={settings} onChange={setSettings} />
}
```

使用懒初始化来计算初始值，如从 localStorage/sessionStorage、构建数据结构（索引、映射）、读取 DOM、或进行重变换。

对于简单原始值 (`useState(0)`), 直接引用 (`useState(props.value)`), 或便宜的字面量 (`useState({})`), 函数形式是不必要的。

### 5.11 使用 Transitions 对非紧急更新

**影响：中 (维持 UI 响应性)**

标记频繁、非紧急状态更新为 transitions，以维持 UI 响应性。

**错误：每次滚动都阻塞 UI**

```tsx
function ScrollTracker() {
  const [scrollY, setScrollY] = useState(0)
  useEffect(() => {
    const handler = () => setScrollY(window.scrollY)
    window.addEventListener('scroll', handler, { passive: true })
    return () => window.removeEventListener('scroll', handler)
  }, [])
}
```

**正确：非阻塞更新**

```tsx
import { startTransition } from 'react'

function ScrollTracker() {
  const [scrollY, setScrollY] = useState(0)
  useEffect(() => {
    const handler = () => {
      startTransition(() => setScrollY(window.scrollY))
    }
    window.addEventListener('scroll', handler, { passive: true })
    return () => window.removeEventListener('scroll', handler)
  }, [])
}
```

### 5.12 使用 useRef 对临时值

**影响：中 (避免频繁更新时不必要的重渲染)**

当值频繁变化且你不想在每次更新时都重新渲染（例如，鼠标追踪器、间隔、临时标志）时，将它存入 `useRef` 而非 `useState`。保持组件状态用于 UI；使用 refs 用于临时 DOM 相邻值。更新 ref 不会触发重渲染。

**错误：每次更新都重新渲染**

```tsx
function Tracker() {
  const [lastX, setLastX] = useState(0)

  useEffect(() => {
    const onMove = (e: MouseEvent) => setLastX(e.clientX)
    window.addEventListener('mousemove', onMove)
    return () => window.removeEventListener('mousemove', onMove)
  }, [])

  return (
    <div
      style={{
        position: 'fixed',
        top: 0,
        left: lastX,
        width: 8,
        height: 8,
        background: 'black',
      }}
    />
  )
}
```

**正确：无重渲染**

```tsx
function Tracker() {
  const lastXRef = useRef(0)
  const dotRef = useRef<HTMLDivElement>(null)

  useEffect(() => {
    const onMove = (e: MouseEvent) => {
      lastXRef.current = e.clientX
      const node = dotRef.current
      if (node) {
        node.style.transform = `translateX(${e.clientX}px)`
      }
    }
    window.addEventListener('mousemove', onMove)
    return () => window.removeEventListener('mousemove', onMove)
  }, [])

  return (
    <div
      ref={dotRef}
      style={{
        position: 'fixed',
        top: 0,
        left: 0,
        width: 8,
        height: 8,
        background: 'black',
        transform: 'translateX(0px)',
      }}
    />
  )
}
```

---

## 6. 渲染性能

**影响：中**

优化渲染过程减少浏览器需要做的工作。

### 6.1 动画应包裹 div 而非 SVG 元素

**影响：低 (启用硬件加速)**

许多浏览器没有硬件加速对 CSS3 动画在 SVG 元素上的支持。将 SVG 包裹在 `<div>` 中并动画 wrapper。

**错误：直接动画 SVG - 无硬件加速**

```tsx
function LoadingSpinner() {
  return (
    <svg 
      className="animate-spin"
      width="24" 
      height="24" 
      viewBox="0 0 24 24"
    >
      <circle cx="12" cy="12" r="10" stroke="currentColor" />
    </svg>
  )
}
```

**正确：动画 wrapper div - 硬件加速**

```tsx
function LoadingSpinner() {
  return (
    <div className="animate-spin">
      <svg 
        width="24" 
        height="24" 
        viewBox="0 0 24 24"
      >
        <circle cx="12" cy="12" r="10" stroke="currentColor" />
      </svg>
    </div>
  )
}
```

这适用于所有 CSS 变换和过渡 (`transform`, `opacity`, `translate`, `scale`, `rotate`). Wrapper div 允许浏览器使用 GPU 加速进行更平滑的动画。

### 6.2 CSS content-visibility for Long Lists

**影响：高 (更快的初始渲染)**

应用 `content-visibility: auto` 来延迟 off-screen 渲染。

**CSS:**

```css
.message-item {
  content-visibility: auto;
  contain-intrinsic-size: 0 80px;
}
```

**例子：**

```tsx
function MessageList({ messages }: { messages: Message[] }) {
  return (
    <div className="overflow-y-auto h-screen">
      {messages.map(msg => (
        <div key={msg.id} className="message-item">
          <Avatar user={msg.author} />
          <div>{msg.content}</div>
        </div>
      ))}
    </div>
  )
}
```

对于 1000 条消息，浏览器跳过 ~990 个 off-screen 项目（10× 更快的初始渲染）。

### 6.3 Hoist Static JSX Elements

**影响：低 (避免重新创建)**

提取静态 JSX 外部组件，避免重新创建。

**错误：每次渲染都重新创建**

```tsx
function LoadingSkeleton() {
  return <div className="animate-pulse h-20 bg-gray-200" />
}

function Container() {
  return (
    <div>
      {loading && <LoadingSkeleton />}
    </div>
  )
}
```

**正确：重用相同元素**

```tsx
const loadingSkeleton = (
  <div className="animate-pulse h-20 bg-gray-200" />
)

function Container() {
  return (
    <div>
      {loading && loadingSkeleton}
    </div>
  )
}
```

这尤其有助于大型和静态 SVG 节点，它们在每次渲染时都可能很昂贵。

**注意：** 如果你的项目有 [React Compiler](https://react.dev/learn/react-compiler) 启用，编译器自动优化静态 JSX 元素和组件重新渲染，使得手动 hoisting 不必要。

### 6.4 优化 SVG 精度

**影响：低 (减少文件大小)**

减少 SVG 坐标精度以减小文件大小。最优精度取决于 viewBox 大小，但一般减少精度是值得考虑的。

**错误：过度精度**

```svg
<path d="M 10.293847 20.847362 L 30.938472 40.192837" />
```

**正确：1 位小数**

```svg
<path d="M 10.3 20.8 L 30.9 40.2" />
```

**自动化：SVGO**

```bash
npx svgo --precision=1 --multipass icon.svg
```

### 6.5 防止水合不匹配而无闪烁

**影响：中 (避免视觉闪烁和水合错误)**

当渲染依赖于客户端存储（localStorage, cookies）时，避免 SSR 破坏和 post-hydration 闪烁，通过注入一个同步脚本在 React 水合之前更新 DOM。

**错误：SSR 破坏**

```tsx
function ThemeWrapper({ children }: { children: ReactNode }) {
  // localStorage is not available on server - throws error
  const theme = localStorage.getItem('theme') || 'light'
  
  return (
    <div className={theme}>
      {children}
    </div>
  )
}
```

SSR 会失败，因为 `localStorage` 是 undefined。

**错误：视觉闪烁**

```tsx
function ThemeWrapper({ children }: { children: ReactNode }) {
  const [theme, setTheme] = useState('light')
  
  useEffect(() => {
    // Runs after hydration - causes visible flash
    const stored = localStorage.getItem('theme')
    if (stored) {
      setTheme(stored)
    }
  }, [])
  
  return (
    <div className={theme}>
      {children}
    </div>
  )
}
```

组件首先渲染为默认值 (`light`), 然后在水合后更新，导致可见的错误内容。

**正确：无闪烁，无水合不匹配**

```tsx
function ThemeWrapper({ children }: { children: ReactNode }) {
  return (
    <>
      <div id="theme-wrapper">
        {children}
      </div>
      <script
        dangerouslySetInnerHTML={{
          __html: `
            (function() {
              try {
                var theme = localStorage.getItem('theme') || 'light';
                var el = document.getElementById('theme-wrapper');
                if (el) el.className = theme;
              } catch (e) {}
            })();
          `,
        }}
      />
    </>
  )
}
```

The inline script executes synchronously before showing the element, ensuring the DOM already has the correct value. No flickering, no hydration mismatch.

This pattern is especially useful for theme toggles, user preferences, authentication states, and any client-only data that should render immediately without flashing default values.

### 6.6 抑制预期水合警告

**影响：低-中 (避免已知差异的噪音警告)**

在 SSR 框架（如 Next.js）中，一些值在服务器和客户端上是故意不同的（随机 ID、日期、时区格式化）。对于这些 *预期* 不匹配，用 `suppressHydrationWarning` 包裹动态文本以防止噪音警告。不要用这个来隐藏真实错误。不要过度使用。

**错误：已知差异警告**

```tsx
function Timestamp() {
  return <span>{new Date().toLocaleString()}</span>
}
```

**正确：抑制预期差异**

```tsx
function Timestamp() {
  return (
    <span suppressHydrationWarning>
      {new Date().toLocaleString()}
    </span>
  )
}
```

### 6.7 使用 Activity 组件来显示/隐藏

**影响：中 (保存状态/DOM)**

使用 React 的 `<Activity>` 来保存昂贵组件的 state/DOM。

**用法：**

```tsx
import { Activity } from 'react'

function Dropdown({ isOpen }: Props) {
  return (
    <Activity mode={isOpen ? 'visible' : 'hidden'}>
      <ExpensiveMenu />
    </Activity>
  )
}
```

避免昂贵的重新渲染和状态丢失。

### 6.8 使用显式条件渲染

**影响：低 (防止渲染 0 或 NaN)**

使用显式三元表达式 (`? :`) 而非 `&&` 进行条件渲染，当条件可以是 `0`、`NaN` 或其他假值时。

**错误：当 count 是 0 时渲染 "0"**

```tsx
function Badge({ count }: { count: number }) {
  return (
    <div>
      {count && <span className="badge">{count}</span>}
    </div>
  )
}

// 当 count = 0 时，渲染: <div>0</div>
// 当 count = 5 时，渲染: <div><span class="badge">5</span></div>
```

**正确：当 count 是 0 时渲染为空**

```tsx
function Badge({ count }: { count: number }) {
  return (
    <div>
      {count > 0 ? <span className="badge">{count}</span> : null}
    </div>
  )
}

// 当 count = 0 时，渲染: <div></div>
// 当 count = 5 时，渲染: <div><span class="badge">5</span></div>
```

### 6.9 使用 useTransition 而非手动 loading 状态

**影响：低 (减少 re-renders 和改进代码清晰度)**

使用 `useTransition` 而非手动 `useState` 来处理 loading 状态。这提供了内置的 `isPending` 状态和自动管理 transitions。

**错误：手动 loading 状态**

```tsx
function SearchResults() {
  const [query, setQuery] = useState('')
  const [results, setResults] = useState([])
  const [isLoading, setIsLoading] = useState(false)

  const handleSearch = async (value: string) => {
    setIsLoading(true)
    setQuery(value)
    const data = await fetchResults(value)
    setResults(data)
    setIsLoading(false)
  }

  return (
    <>
      <input onChange={(e) => handleSearch(e.target.value)} />
      {isLoading && <Spinner />}
      <ResultsList results={results} />
    </>
  )
}
```

**正确：useTransition 带内置 pending 状态**

```tsx
import { useTransition, useState } from 'react'

function SearchResults() {
  const [query, setQuery] = useState('')
  const [results, setResults] = useState([])
  const [isPending, startTransition] = useTransition()

  const handleSearch = (value: string) => {
    setQuery(value) // Update input immediately
    
    startTransition(async () => {
      // Fetch and update results
      const data = await fetchResults(value)
      setResults(data)
    })
  }

  return (
    <>
      <input onChange={(e) => handleSearch(e.target.value)} />
      {isPending && <Spinner />}
      <ResultsList results={results} />
    </>
  )
}
```

**好处：**

- **Automatic pending state**: No need to manually manage `setIsLoading(true/false)`

- **Error resilience**: Pending state correctly resets even if the transition throws

- **Better responsiveness**: Keeps the UI responsive during updates

- **Interrupt handling**: New transitions automatically cancel pending ones

Reference: [https://react.dev/reference/react/useTransition](https://react.dev/reference/react/useTransition)

---

## 7. JavaScript 性能

**影响：中低**

微优化可以带来有意义的改进。

### 7.1 避免布局抖动

**影响：中 (防止强制同步布局和减少性能瓶颈)**

避免交错样式写入和布局读取。当你在样式变化之间读取布局属性（如 `offsetWidth`、`getBoundingClientRect()`、`getComputedStyle()`）时，浏览器被迫触发同步重流。

**这可以：浏览器批量样式变化**

```typescript
function updateElementStyles(element: HTMLElement) {
  // 每行无效化样式，但浏览器批量重新计算
  element.style.width = '100px'
  element.style.height = '200px'
  element.style.backgroundColor = 'blue'
  element.style.border = '1px solid black'
}
```

**错误：交错读写导致重流**

```typescript
function layoutThrashing(element: HTMLElement) {
  element.style.width = '100px'
  const width = element.offsetWidth  // 强制重流
  element.style.height = '200px'
  const height = element.offsetHeight  // 强制另一个重流
}
```

**正确：批量写入，然后读取一次**

```typescript
function updateElementStyles(element: HTMLElement) {
  // 批量所有写入
  element.style.width = '100px'
  element.style.height = '200px'
  element.style.backgroundColor = 'blue'
  element.style.border = '1px solid black'
  
  // 读取一次
  const { width, height } = element.getBoundingClientRect()
}
```

**正确：批量读取，然后写入**

```typescript
function updateElementStyles(element: HTMLElement) {
  element.classList.add('highlighted-box')
  
  const { width, height } = element.getBoundingClientRect()
}
```

**更好：使用 CSS 类**

**React 例子：**

```tsx
// 错误：交错样式变化和布局查询
function Box({ isHighlighted }: { isHighlighted: boolean }) {
  const ref = useRef<HTMLDivElement>(null)
  
  useEffect(() => {
    if (ref.current && isHighlighted) {
      ref.current.style.width = '100px'
      const width = ref.current.offsetWidth // 强制布局
      ref.current.style.height = '200px'
    }
  }, [isHighlighted])
  
  return <div ref={ref}>Content</div>
}

// 正确：切换 class
function Box({ isHighlighted }: { isHighlighted: boolean }) {
  return (
    <div className={isHighlighted ? 'highlighted-box' : ''}>
      Content
    </div>
  )
}
```

优先使用 CSS 类而不是 inline styles。CSS 文件被浏览器缓存，类提供更好的分离关注和更容易维护。

见 [this gist](https://gist.github.com/paulirish/5d52fb081b3570c81e3a) 和 [CSS Triggers](https://csstriggers.com/) 获取更多关于布局强制操作的信息。

### 7.2 构建索引 Map 用于重复查找

**影响：中低 (1M ops to 2K ops)**

多个 `.find()` 调用应使用 Map。

**错误 (O(n) 每次查找):**

```typescript
function processOrders(orders: Order[], users: User[]) {
  return orders.map(order => ({
    ...order,
    user: users.find(u => u.id === order.userId)
  }))
}
```

**正确 (O(1) 每次查找):**

```typescript
function processOrders(orders: Order[], users: User[]) {
  const userById = new Map(users.map(u => [u.id, u]))

  return orders.map(order => ({
    ...order,
    user: userById.get(order.userId)
  }))
}
```

构建 map 一次 (O(n)), 然后所有查找都是 O(1)。

对于 1000 个订单 × 1000 个用户：1M ops → 2K ops。

### 7.3 缓存属性访问在循环中

**影响：中低 (减少查找)**

在热点路径中缓存对象属性查找。

**错误：3 查找 × N 迭代**

```typescript
for (let i = 0; i < arr.length; i++) {
  process(obj.config.settings.value)
}
```

**正确：1 查找**

```typescript
const value = obj.config.settings.value
const len = arr.length
for (let i = 0; i < len; i++) {
  process(value)
}
```

### 7.4 缓存重复函数调用

**影响：中 (避免冗余计算)**

使用模块级 Map 缓存函数结果，当同一函数在渲染期间多次调用时。

**错误：冗余计算**

```tsx
function ProjectList({ projects }: { projects: Project[] }) {
  return (
    <div>
      {projects.map(project => {
        // slugify() 调用 100+ 次
        const slug = slugify(project.name)
        
        return <ProjectCard key={project.id} slug={slug} />
      })}
    </div>
  )
}
```

**正确：缓存结果**

```tsx
// 模块级缓存
const slugifyCache = new Map<string, string>()

function cachedSlugify(text: string): string {
  if (slugifyCache.has(text)) {
    return slugifyCache.get(text)!
  }
  const result = slugify(text)
  slugifyCache.set(text, result)
  return result
}

function ProjectList({ projects }: { projects: Project[] }) {
  return (
    <div>
      {projects.map(project => {
        // 计算一次
        const slug = cachedSlugify(project.name)
        
        return <ProjectCard key={project.id} slug={slug} />
      })}
    </div>
  )
}
```

**单值函数的简化模式：**

```typescript
let isLoggedInCache: boolean | null = null

function isLoggedIn(): boolean {
  if (isLoggedInCache !== null) {
    return isLoggedInCache
  }
  
  isLoggedInCache = document.cookie.includes('auth=')
  return isLoggedInCache
}

// Clear cache when auth changes
function onAuthChange() {
  isLoggedInCache = null
}
```

使用 Map (而不是 hook) 所以它在任何地方都有效：工具、事件处理器，不只是 React 组件。

参考：[https://vercel.com/blog/how-we-made-the-vercel-dashboard-twice-as-fast](https://vercel.com/blog/how-we-made-the-vercel-dashboard-twice-as-fast)

### 7.5 缓存 Storage API 调用

**影响：中低 (减少昂贵 I/O)**

`localStorage`, `sessionStorage`, 和 `document.cookie` 是同步且昂贵的。缓存读取在内存中。

**错误：每次调用都读取存储**

```typescript
function getTheme() {
  return localStorage.getItem('theme') ?? 'light'
}
// 调用 10 次 = 10 次存储读取
```

**正确：Map 缓存**

```typescript
const storageCache = new Map<string, string | null>()

function getLocalStorage(key: string) {
  if (!storageCache.has(key)) {
    storageCache.set(key, localStorage.getItem(key))
  }
  return storageCache.get(key)
}

function setLocalStorage(key: string, value: string) {
  localStorage.setItem(key, value)
  storageCache.set(key, value)  // keep cache in sync
}
```

使用 Map (而不是 hook) 所以它在任何地方都有效：工具、事件处理器，不只是 React 组件。

**Cookie 缓存：**

```typescript
let cookieCache: Record<string, string> | null = null

function getCookie(name: string) {
  if (!cookieCache) {
    cookieCache = Object.fromEntries(
      document.cookie.split('; ').map(c => c.split('='))
    )
  }
  return cookieCache[name]
}
```

**重要：在外部变化时无效化**

```typescript
window.addEventListener('storage', (e) => {
  if (e.key) storageCache.delete(e.key)
})

document.addEventListener('visibilitychange', () => {
  if (document.visibilityState === 'visible') {
    storageCache.clear()
  }
})
```

如果存储可以被外部更改（另一个标签，服务器设置的 cookies），无效化缓存：

### 7.6 合并多次数组迭代

**影响：中低 (减少迭代)**

多个 `.filter()` 或 `.map()` 调用迭代数组多次。合并为一个循环。

**错误：3 次迭代**

```typescript
const admins = users.filter(u => u.isAdmin)
const testers = users.filter(u => u.isTester)
const inactive = users.filter(u => !u.isActive)
```

**正确：1 次迭代**

```typescript
const admins: User[] = []
const testers: User[] = []
const inactive: User[] = []

for (const user of users) {
  if (user.isAdmin) admins.push(user)
  if (user.isTester) testers.push(user)
  if (!user.isActive) inactive.push(user)
}
```

### 7.7 早期长度检查数组比较

**影响：中高 (避免昂贵操作当长度不同时)**

当比较数组时，如果长度不同，它们不可能相等。

在真实世界中，这在热点路径（事件处理器、渲染循环）中尤其有价值。

**错误：总是运行昂贵比较**

```typescript
function hasChanges(current: string[], original: string[]) {
  // 总是排序和连接，即使长度不同
  return current.sort().join() !== original.sort().join()
}
```

两个 O(n log n) 排序运行，即使 `current.length` 是 5，`original.length` 是 100。还有连接数组和比较字符串的开销。

**正确 (O(1) 长度检查首先):**

```typescript
function hasChanges(current: string[], original: string[]) {
  // 早期返回如果长度不同
  if (current.length !== original.length) {
    return true
  }
  // 只有在长度匹配时才排序
  const currentSorted = current.toSorted()
  const originalSorted = original.toSorted()
  for (let i = 0; i < currentSorted.length; i++) {
    if (currentSorted[i] !== originalSorted[i]) {
      return true
    }
  }
  return false
}
```

这个新方法更高效，因为：

- 它避免了当长度不同时排序和连接数组的开销

- 它避免了消耗内存为连接的字符串（尤其是大数组）

- 它避免了突变原数组

- 它在找到差异时返回早

### 7.8 早期 return from Functions

**影响：中低 (避免不必要的计算)**

在结果确定时返回，以避免不必要的处理。

**错误：处理所有项目，即使找到答案**

```typescript
function validateUsers(users: User[]) {
  let hasError = false
  let errorMessage = ''
  
  for (const user of users) {
    if (!user.email) {
      hasError = true
      errorMessage = 'Email required'
    }
    if (!user.name) {
      hasError = true
      errorMessage = 'Name required'
    }
    // 继续检查所有用户，即使找到错误
  }
  
  return hasError ? { valid: false, error: errorMessage } : { valid: true }
}
```

**正确：立即返回第一个错误**

```typescript
function validateUsers(users: User[]) {
  for (const user of users) {
    if (!user.email) {
      return { valid: false, error: 'Email required' }
    }
    if (!user.name) {
      return { valid: false, error: 'Name required' }
    }
  }

  return { valid: true }
}
```

### 7.9 Hoist RegExp Creation

**影响：中低 (避免重建)**

不要在渲染中创建 RegExp。上提至模块范围或用 `useMemo()` 缓存。

**错误：每次渲染都创建 RegExp**

```tsx
function Highlighter({ text, query }: Props) {
  const regex = new RegExp(`(${query})`, 'gi')
  const parts = text.split(regex)
  return <>{parts.map((part, i) => ...)}</>
}
```

**正确：缓存或上提**

```tsx
const EMAIL_REGEX = /^[^\s@]+@[^\s@]+\.[^\s@]+$/

function Highlighter({ text, query }: Props) {
  const regex = useMemo(
    () => new RegExp(`(${escapeRegex(query)})`, 'gi'),
    [query]
  )
  const parts = text.split(regex)
  return <>{parts.map((part, i) => ...)}</>
}
```

**警告：global regex 有 mutable state**

```typescript
const regex = /foo/g
regex.test('foo')  // true, lastIndex = 3
regex.test('foo')  // false, lastIndex = 0
```

全局 regex (`/g`) 有 mutable `lastIndex` 状态：

### 7.10 Use Loop for Min/Max Instead of Sort

**影响：低 (O(n) 而非 O(n log n))**

找到最小或最大的元素只需要单次遍历。排序是浪费且慢的。

**错误 (O(n log n) - sort to find latest):**

```typescript
interface Project {
  id: string
  name: string
  updatedAt: number
}

function getLatestProject(projects: Project[]) {
  const sorted = [...projects].sort((a, b) => b.updatedAt - a.updatedAt)
  return sorted[0]
}
```

排序整个数组，只为找到最大值。

**错误 (O(n log n) - sort for oldest and newest):**

```typescript
function getOldestAndNewest(projects: Project[]) {
  const sorted = [...projects].sort((a, b) => a.updatedAt - b.updatedAt)
  return { oldest: sorted[0], newest: sorted[sorted.length - 1] }
}
```

仍然在不需要时排序。

**正确 (O(n) - 单次遍历):**

```typescript
function getLatestProject(projects: Project[]) {
  if (projects.length === 0) return null
  
  let latest = projects[0]
  
  for (let i = 1; i < projects.length; i++) {
    if (projects[i].updatedAt > latest.updatedAt) {
      latest = projects[i]
    }
  }
  
  return latest
}

function getOldestAndNewest(projects: Project[]) {
  if (projects.length === 0) return { oldest: null, newest: null }
  
  let oldest = projects[0]
  let newest = projects[0]
  
  for (let i = 1; i < projects.length; i++) {
    if (projects[i].updatedAt < oldest.updatedAt) oldest = projects[i]
    if (projects[i].updatedAt > newest.updatedAt) newest = projects[i]
  }
  
  return { oldest, newest }
}
```

单次遍历，无复制，无排序。

**替代方案：Math.min/Math.max 对于小数组**

```typescript
const numbers = [5, 2, 8, 1, 9]
const min = Math.min(...numbers)
const max = Math.max(...numbers)
```

这在小数组中有效，但对非常大的数组可能较慢或抛出错误，因为 spread operator 限制。最大数组长度在 Chrome 143 中约为 124000，在 Safari 18 中约为 638000；确切数字可能有所不同 - 见 [the fiddle](https://jsfiddle.net/qw1jabsx/4/). 使用循环方法更可靠。

### 7.11 Use Set/Map for O(1) Lookups

**影响：中低 (O(n) 到 O(1))**

将数组转换为 Set/Map 用于重复成员检查。

**错误 (O(n) 每次检查):**

```typescript
const allowedIds = ['a', 'b', 'c', ...]
items.filter(item => allowedIds.includes(item.id))
```

**正确 (O(1) 每次检查):**

```typescript
const allowedIds = new Set(['a', 'b', 'c', ...])
items.filter(item => allowedIds.has(item.id))
```

### 7.12 Use toSorted() Instead of sort() for Immutability

**影响：中高 (防止 React 状态中的突变 bugs)**

`.sort()` 在原地突变，这可能导致 React 状态和 props 的 bugs。使用 `.toSorted()` 创建一个新排序数组，而不突变。

**错误：突变原数组**

```typescript
function UserList({ users }: { users: User[] }) {
  // 突变 users prop array!
  const sorted = useMemo(
    () => users.sort((a, b) => a.name.localeCompare(b.name)),
    [users]
  )
  return <div>{sorted.map(renderUser)}</div>
}
```

**正确：创建新数组**

```typescript
function UserList({ users }: { users: User[] }) {
  // 创建新排序数组，原数组不变
  const sorted = useMemo(
    () => users.toSorted((a, b) => a.name.localeCompare(b.name)),
    [users]
  )
  return <div>{sorted.map(renderUser)}</div>
}
```

**为什么这在 React 中重要：**

1. Props/state 突变破坏 React 的不可变模型 - React 期望 props 和 state 是只读的

2. 导致 stale closure bugs - 在闭包中突变数组（回调、effect）会导致意外行为

**浏览器支持：旧浏览器的 fallback**

```typescript
// Fallback for older browsers
const sorted = [...items].sort((a, b) => a.value - b.value)
```

`.toSorted()` 在所有现代浏览器中可用（Chrome 110+, Safari 16+, Firefox 115+, Node.js 20+）。对于旧环境，使用 spread operator：

**其他不可变数组方法：**

- `.toSorted()` - 不可变排序

- `.toReversed()` - 不可变反转

- `.toSpliced()` - 不可变 splice

- `.with()` - 不可变元素替换

---

## 8. 高级模式

**影响：低**

高级模式用于特定情况，需要仔细实现。

### 8.1 应用仅初始化一次

**影响：低-中 (避免开发中重复初始化)**

不要将必须运行一次 per app load 的应用级初始化放在 `useEffect([])` 的组件中。组件可以重新挂载，effect 会重新运行。使用模块级守卫或入口模块的顶层初始化。

**错误：在开发中运行两次，重新挂载时重新运行**

```tsx
function Comp() {
  useEffect(() => {
    loadFromStorage()
    checkAuthToken()
  }, [])

  // ...
}
```

**正确：一次 per app load**

```tsx
let didInit = false

function Comp() {
  useEffect(() => {
    if (didInit) return
    didInit = true
    loadFromStorage()
    checkAuthToken()
  }, [])

  // ...
}
```

参考：[https://react.dev/learn/you-might-not-need-an-effect#initializing-the-application](https://react.dev/learn/you-might-not-need-an-effect#initializing-the-application)

### 8.2 存储事件处理器在 refs 中

**影响：低 (稳定订阅)**

当用于不应在回调变化时重新订阅的 effect 中，存储回调在 refs 中。

**错误：每次渲染都重新订阅**

```tsx
function useWindowEvent(event: string, handler: (e) => void) {
  useEffect(() => {
    window.addEventListener(event, handler)
    return () => window.removeEventListener(event, handler)
  }, [event, handler])
}
```

**正确：稳定订阅**

```tsx
import { useEffectEvent } from 'react'

function useWindowEvent(event: string, handler: (e) => void) {
  const onEvent = useEffectEvent(handler)

  useEffect(() => {
    window.addEventListener(event, onEvent)
    return () => window.removeEventListener(event, onEvent)
  }, [event])
}
```

**替代方案：如果你在最新 React 上：**

`useEffectEvent` 提供了更干净的 API，用于相同模式：它创建一个稳定的函数引用，总是调用最新的 handler。

### 8.3 useEffectEvent for Stable Callback Refs

**影响：低 (防止 effect 重新运行)**

访问最新值而不添加到依赖数组。防止 effect 重新运行，同时避免 stale closures。

**错误：effect 重新运行在每次回调变化时**

```tsx
function SearchInput({ onSearch }: { onSearch: (q: string) => void }) {
  const [query, setQuery] = useState('')

  useEffect(() => {
    const timeout = setTimeout(() => onSearch(query), 300)
    return () => clearTimeout(timeout)
  }, [query, onSearch])
}
```

**正确：使用 React 的 useEffectEvent**

```tsx
import { useEffectEvent } from 'react';

function SearchInput({ onSearch }: { onSearch: (q: string) => void }) {
  const [query, setQuery] = useState('')
  const onSearchEvent = useEffectEvent(onSearch)

  useEffect(() => {
    const timeout = setTimeout(() => onSearchEvent(query), 300)
    return () => clearTimeout(timeout)
  }, [query])
}
```

---

## 参考

1. [https://react.dev](https://react.dev)
2. [https://nextjs.org](https://nextjs.org)
3. [https://swr.vercel.app](https://swr.vercel.app)
4. [https://github.com/shuding/better-all](https://github.com/shuding/better-all)
5. [https://github.com/isaacs/node-lru-cache](https://github.com/isaacs/node-lru-cache)
6. [https://vercel.com/blog/how-we-optimized-package-imports-in-next-js](https://vercel.com/blog/how-we-optimized-package-imports-in-next-js)
7. [https://vercel.com/blog/how-we-made-the-vercel-dashboard-twice-as-fast](https://vercel.com/blog/how-we-made-the-vercel-dashboard-twice-as-fast)
