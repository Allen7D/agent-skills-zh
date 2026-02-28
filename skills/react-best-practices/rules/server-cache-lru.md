---
title: 跨请求 LRU 缓存
impact: HIGH
impactDescription: 跨请求缓存
tags: server, cache, lru, cross-request
---

## 跨请求 LRU 缓存

`React.cache()` 只在单个请求内有效。对于需要在连续请求间共享的数据（如用户连续点击多个端点都需要同一数据），请使用 LRU 缓存。

**实现示例：**

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

// 请求1：数据库查询，结果缓存
// 请求2：缓存命中，无需数据库查询
```

适用于用户在几秒内连续操作多个端点且需要相同数据的场景。

**在 Vercel [Fluid Compute](https://vercel.com/docs/fluid-compute) 下：** LRU 缓存尤其有效，因为多个并发请求可以共享同一个函数实例和缓存。这意味着缓存可跨请求持久，无需 Redis 等外部存储。

**在传统 serverless 下：** 每次调用都是隔离的，建议用 Redis 等跨进程缓存。

参考：[https://github.com/isaacs/node-lru-cache](https://github.com/isaacs/node-lru-cache)
