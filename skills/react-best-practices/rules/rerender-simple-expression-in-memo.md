---
title: 不要将简单原始类型表达式包裹在 useMemo 中
impact: LOW-MEDIUM
impactDescription: 每次渲染都浪费计算
tags: rerender, useMemo, optimization
---

## 不要将简单原始类型表达式包裹在 useMemo 中

当表达式很简单（只有少量逻辑或算术运算）且结果类型为原始类型（布尔、数字、字符串）时，不要用 `useMemo` 包裹。
调用 `useMemo` 并比较依赖项可能比表达式本身消耗更多资源。

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
