---
title: 函数的早期返回
impact: 低-中等
impactDescription: 避免不必要的计算
tags: javascript, functions, optimization, early-return
---

## 函数的早期返回

在结果确定时早期返回，跳过不必要的处理。

**错误示例（即使在找到答案后仍然处理所有项目）：**

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
    // 即使在发现错误后仍继续检查所有用户
  }
  
  return hasError ? { valid: false, error: errorMessage } : { valid: true }
}
```

**正确示例（在第一个错误时立即返回）：**

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
