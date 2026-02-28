---
title: 提升静态 JSX 元素
impact: 低
impactDescription: 避免重复创建
tags: rendering, jsx, static, optimization
---

## 提升静态 JSX 元素

将静态 JSX 提取到组件外以避免重新创建。

**错误示例（每次渲染都重新创建元素）：**

```tsx
function LoadingSkeleton() {
  return <div className="animate-pulse h-20 bg-gray-200" />
}

function Container() {
  return (
    <div>
      {loading && <LoadingSkeleton />}
    </div>
  )
}
```

**正确示例（重用相同元素）：**

```tsx
const loadingSkeleton = (
  <div className="animate-pulse h-20 bg-gray-200" />
)

function Container() {
  return (
    <div>
      {loading && loadingSkeleton}
    </div>
  )
}
```

这对于大型和静态的 SVG 节点特别有用，因为在每次渲染时重新创建它们可能会很昂贵。

**注意：** 如果您的项目启用了 [React Compiler](https://react.dev/learn/react-compiler)，编译器会自动提升静态 JSX 元素并优化组件重新渲染，使得手动提升变得不必要。
