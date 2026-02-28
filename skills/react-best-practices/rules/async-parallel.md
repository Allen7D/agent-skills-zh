---
title: 对独立操作使用 Promise.all()
impact: 关键
impactDescription: 2-10× 性能提升
tags: async, parallelization, promises, waterfalls
---

## 对独立操作使用 Promise.all()

当异步操作没有相互依赖时，使用 `Promise.all()` 并发执行它们。

**错误示例（顺序执行，3 轮请求）：**

```typescript
const user = await fetchUser()
const posts = await fetchPosts()
const comments = await fetchComments()
```

**正确示例（并行执行，1 轮请求）：**

```typescript
const [user, posts, comments] = await Promise.all([
  fetchUser(),
  fetchPosts(),
  fetchComments()
])
```
