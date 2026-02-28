---
title: 缩小 Effect 依赖项
impact: 低
impactDescription: 最小化 effect 重新运行
tags: rerender, useEffect, dependencies, optimization
---

## 缩小 Effect 依赖项

指定原始依赖项而不是对象，以最小化 effect 重新运行。

**错误示例（任何用户字段变化时都重新运行）：**

```tsx
useEffect(() => {
  console.log(user.id)
}, [user])
```

**正确示例（只在 id 变化时重新运行）：**

```tsx
useEffect(() => {
  console.log(user.id)
}, [user.id])
```

**对于派生状态，在 effect 外部计算：**

```tsx
// 错误示例：在 width=767, 766, 765... 时运行
useEffect(() => {
  if (width < 768) {
    enableMobileMode()
  }
}, [width])

// 正确示例：只在布尔转换时运行
const isMobile = width < 768
useEffect(() => {
  if (isMobile) {
    enableMobileMode()
  }
}, [isMobile])
```
