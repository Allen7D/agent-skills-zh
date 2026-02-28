---
title: 用 React.cache() 实现单请求去重
impact: MEDIUM
impactDescription: 请求内去重
tags: server, cache, react-cache, deduplication
---

## 用 React.cache() 实现单请求去重

使用 `React.cache()` 进行服务端请求内去重。认证和数据库查询最受益。

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

在单个请求内，多次调用 `getCurrentUser()` 只会执行一次查询。

**避免将内联对象作为参数：**

`React.cache()` 使用浅等性（`Object.is`）判断缓存命中。内联对象每次都是新引用，无法命中缓存。

**错误（总是缓存未命中）：**

```typescript
const getUser = cache(async (params: { uid: number }) => {
  return await db.user.findUnique({ where: { id: params.uid } })
})

// 每次调用都创建新对象，无法命中缓存
getUser({ uid: 1 })
getUser({ uid: 1 })  // 缓存未命中，再次查询
```

**正确（缓存命中）：**

```typescript
const getUser = cache(async (uid: number) => {
  return await db.user.findUnique({ where: { id: uid } })
})

// 原始类型参数用值判断
getUser(1)
getUser(1)  // 缓存命中，返回缓存结果
```

如必须传递对象，请传递同一引用：

```typescript
const params = { uid: 1 }
getUser(params)  // 查询
getUser(params)  // 缓存命中（同一引用）
```

**Next.js 特别说明：**

在 Next.js 中，`fetch` API 已自动扩展为请求内记忆化。相同 URL 和参数的请求会自动去重，所以 `fetch` 不需要 `React.cache()`。但对于其他异步任务，`React.cache()` 依然重要：

- 数据库查询（Prisma、Drizzle 等）
- 重计算
- 认证校验
- 文件系统操作
- 任何非 fetch 的异步任务

用 `React.cache()` 去重这些操作。

参考：[React.cache 文档](https://react.dev/reference/react/cache)
