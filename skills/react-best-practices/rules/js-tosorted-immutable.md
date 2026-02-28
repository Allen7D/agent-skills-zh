---
title: 使用 toSorted() 而非 sort() 保持不可变性
impact: 中等-高
impactDescription: 防止 React 状态中的突变 bug
tags: javascript, arrays, immutability, react, state, mutation
---

## 使用 toSorted() 而非 sort() 保持不可变性

`.sort()` 会就地突变数组，这可能在 React 状态和 props 中导致 bug。使用 `.toSorted()` 创建新的排序数组而不进行突变。

**错误示例（突变原始数组）：**

```typescript
function UserList({ users }: { users: User[] }) {
  // 突变了 users prop 数组！
  const sorted = useMemo(
    () => users.sort((a, b) => a.name.localeCompare(b.name)),
    [users]
  )
  return <div>{sorted.map(renderUser)}</div>
}
```

**正确示例（创建新数组）：**

```typescript
function UserList({ users }: { users: User[] }) {
  // 创建新的排序数组，原始数组不变
  const sorted = useMemo(
    () => users.toSorted((a, b) => a.name.localeCompare(b.name)),
    [users]
  )
  return <div>{sorted.map(renderUser)}</div>
}
```

**为什么这在 React 中很重要：**

1. Props/state 突变破坏了 React 的不可变性模型 - React 期望 props 和 state 被视为只读
2. 导致过时闭包 bug - 在闭包（回调、副作用）内突变数组可能导致意外行为

**浏览器支持（旧浏览器的后备方案）：**

`.toSorted()` 在所有现代浏览器中都可用（Chrome 110+、Safari 16+、Firefox 115+、Node.js 20+）。对于旧环境，使用扩展运算符：

```typescript
// 旧浏览器的后备方案
const sorted = [...items].sort((a, b) => a.value - b.value)
```

**其他不可变数组方法：**

- `.toSorted()` - 不可变排序
- `.toReversed()` - 不可变翻转
- `.toSpliced()` - 不可变拼接
- `.with()` - 不可变元素替换
