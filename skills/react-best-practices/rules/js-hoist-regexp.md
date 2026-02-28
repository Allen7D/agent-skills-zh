---
title: 提升 RegExp 创建
impact: 低-中等
impactDescription: 避免重复创建
tags: javascript, regexp, optimization, memoization
---

## 提升 RegExp 创建

不要在渲染内创建 RegExp。提升到模块作用域或使用 `useMemo()` 缓存。

**错误示例Ｈ每次渲染都新建 RegExp）：**

```tsx
function Highlighter({ text, query }: Props) {
  const regex = new RegExp(`(${query})`, 'gi')
  const parts = text.split(regex)
  return <>{parts.map((part, i) => ...)}</>
}
```

**正确示例（缓存或提升）：**

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

**警告（全局正则表达式有可变状态）：**

全局正则表达式（`/g`）有可变的 `lastIndex` 状态：

```typescript
const regex = /foo/g
regex.test('foo')  // true, lastIndex = 3
regex.test('foo')  // false, lastIndex = 0
```
