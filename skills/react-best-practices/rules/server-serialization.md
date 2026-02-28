---
title: 最小化 RSC 边界的序列化
impact: HIGH
impactDescription: 减少数据传输大小
tags: server, rsc, serialization, props
---

## 最小化 RSC 边界的序列化

React Server/Client 边界会将所有对象属性序列化为字符串，并嵌入到 HTML 响应和后续的 RSC 请求中。这些序列化数据会直接影响页面体积和加载时间，因此**数据大小非常重要**。只传递客户端实际需要的字段。

**错误（序列化全部 50 个字段）：**

```tsx
async function Page() {
  const user = await fetchUser()  // 50 个字段
  return <Profile user={user} />
}

'use client'
function Profile({ user }: { user: User }) {
  return <div>{user.name}</div>  // 实际只用到 1 个字段
}
```

**正确（只序列化 1 个字段）：**

```tsx
async function Page() {
  const user = await fetchUser()
  return <Profile name={user.name} />
}

'use client'
function Profile({ name }: { name: string }) {
  return <div>{name}</div>
}
