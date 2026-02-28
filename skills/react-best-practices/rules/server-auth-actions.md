---
title: 服务端操作需鉴权（如同 API 路由）
impact: CRITICAL
impactDescription: 防止未授权访问服务端变更
tags: server, server-actions, authentication, security, authorization
---

## 服务端操作需鉴权（如同 API 路由）

**影响：致命（防止未授权访问服务端变更）**

Server Actions（带 `"use server"` 的函数）会作为公开端点暴露，就像 API 路由一样。务必**在每个 Server Action 内部**校验认证和授权——不要仅依赖中间件、布局守卫或页面级检查，因为 Server Actions 可以被直接调用。

Next.js 文档明确指出：“应以与公开 API 端点相同的安全标准对待 Server Actions，并校验用户是否有权限执行变更。”

**错误（未做认证校验）：**

```typescript
'use server'

export async function deleteUser(userId: string) {
  // 任何人都能调用！没有认证校验
  await db.user.delete({ where: { id: userId } })
  return { success: true }
}
```

**正确（在 action 内部认证）：**

```typescript
'use server'

import { verifySession } from '@/lib/auth'
import { unauthorized } from '@/lib/errors'

export async function deleteUser(userId: string) {
  // 必须在 action 内部校验认证
  const session = await verifySession()
  
  if (!session) {
    throw unauthorized('必须登录')
  }
  
  // 还要校验授权
  if (session.user.role !== 'admin' && session.user.id !== userId) {
    throw unauthorized('不能删除其他用户')
  }
  
  await db.user.delete({ where: { id: userId } })
  return { success: true }
}
```

**带输入校验的示例：**

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
  // 先校验输入
  const validated = updateProfileSchema.parse(data)
  
  // 再认证
  const session = await verifySession()
  if (!session) {
    throw new Error('未授权')
  }
  
  // 再授权
  if (session.user.id !== validated.userId) {
    throw new Error('只能修改自己的资料')
  }
  
  // 最后执行变更
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
