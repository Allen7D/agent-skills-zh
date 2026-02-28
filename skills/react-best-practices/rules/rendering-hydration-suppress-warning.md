---
title: 抑制预期的水合不匹配警告
impact: 低-中等
impactDescription: 避免已知差异的嘈杂水合警告
tags: rendering, hydration, ssr, nextjs
---

## 抑制预期的水合不匹配警告

在 SSR 框架（如 Next.js）中，有些值在服务器和客户端之间有意地不同（随机ID、日期、本地化/时区格式化）。对于这些*预期的*不匹配，将动态文本包装在带有 `suppressHydrationWarning` 的元素中以防止嘈杂的警告。不要用这个来隐藏真正的 bug。不要过度使用它。

**错误示例（已知不匹配警告）：**

```tsx
function Timestamp() {
  return <span>{new Date().toLocaleString()}</span>
}
```

**正确示例（仅抑制预期的不匹配）：**

```tsx
function Timestamp() {
  return (
    <span suppressHydrationWarning>
      {new Date().toLocaleString()}
    </span>
  )
}
```
