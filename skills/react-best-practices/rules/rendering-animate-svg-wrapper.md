---
title: 动画 SVG 包装器而非 SVG 元素
impact: 低
impactDescription: 启用硬件加速
tags: rendering, svg, css, animation, performance
---

## 动画 SVG 包装器而非 SVG 元素

许多浏览器没有对 SVG 元素上的 CSS3 动画进行硬件加速。将 SVG 包装在 `<div>` 中，并动画包装器而不是 SVG 本身。

**错误示例（直接动画 SVG - 无硬件加速）：**

```tsx
function LoadingSpinner() {
  return (
    <svg 
      className="animate-spin"
      width="24" 
      height="24" 
      viewBox="0 0 24 24"
    >
      <circle cx="12" cy="12" r="10" stroke="currentColor" />
    </svg>
  )
}
```

**正确示例（动画包装器 div - 硬件加速）：**

```tsx
function LoadingSpinner() {
  return (
    <div className="animate-spin">
      <svg 
        width="24" 
        height="24" 
        viewBox="0 0 24 24"
      >
        <circle cx="12" cy="12" r="10" stroke="currentColor" />
      </svg>
    </div>
  )
}
```

这适用于所有 CSS 变换和过渡（`transform`、`opacity`、`translate`、`scale`、`rotate`）。包装器 div 允许浏览器使用 GPU 加速来实现更平滑的动画。
