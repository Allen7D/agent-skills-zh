---
title: 订阅派生状态
impact: MEDIUM
impactDescription: 降低重渲染频率
tags: rerender, derived-state, media-query, optimization
---

## 订阅派生状态

订阅派生布尔状态而不是连续值，以减少重渲染频率。

**错误（每像素变化都重渲染）：**

```tsx
function Sidebar() {
  const width = useWindowWidth()  // 持续更新
  const isMobile = width < 768
  return <nav className={isMobile ? 'mobile' : 'desktop'} />
}
```

**正确（仅布尔变化时重渲染）：**

```tsx
function Sidebar() {
  const isMobile = useMediaQuery('(max-width: 767px)')
  return <nav className={isMobile ? 'mobile' : 'desktop'} />
}
```
