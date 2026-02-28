---
title: 避免布局抖动
impact: 中等
impactDescription: 防止强制同步布局，减少性能瓶颈
tags: javascript, dom, css, performance, reflow, layout-thrashing
---

## 避免布局抖动

避免在样式更改之间穿插布局读取。当您在样式更改之间读取布局属性（如 `offsetWidth`、`getBoundingClientRect()` 或 `getComputedStyle()`）时，浏览器被迫触发同步重排。

**这样做是可以的（浏览器批量处理样式更改）：**
```typescript
function updateElementStyles(element: HTMLElement) {
  // 每一行都会使样式无效，但浏览器会批量重新计算
  element.style.width = '100px'
  element.style.height = '200px'
  element.style.backgroundColor = 'blue'
  element.style.border = '1px solid black'
}
```

**错误示例（穿插的读取和写入强制重排）：**
```typescript
function layoutThrashing(element: HTMLElement) {
  element.style.width = '100px'
  const width = element.offsetWidth  // Forces reflow
  element.style.height = '200px'
  const height = element.offsetHeight  // 强制另一次重排
}
```

**正确示例（批量写入，然后读取一次）：**
```typescript
function updateElementStyles(element: HTMLElement) {
  // 将所有写入批量处理
  element.style.width = '100px'
  element.style.height = '200px'
  element.style.backgroundColor = 'blue'
  element.style.border = '1px solid black'
  
  // 所有写入完成后读取（单次重排）
  const { width, height } = element.getBoundingClientRect()
}
```

**正确示例（批量读取，然后写入）：**
```typescript
function avoidThrashing(element: HTMLElement) {
  // 读取阶段 - 首先进行所有布局查询
  const rect1 = element.getBoundingClientRect()
  const offsetWidth = element.offsetWidth
  const offsetHeight = element.offsetHeight
  
  // 写入阶段 - 所有样式更改之后
  element.style.width = '100px'
  element.style.height = '200px'
}
```

**更好：使用 CSS 类**
```css
.highlighted-box {
  width: 100px;
  height: 200px;
  background-color: blue;
  border: 1px solid black;
}
```
```typescript
function updateElementStyles(element: HTMLElement) {
  element.classList.add('highlighted-box')
  
  const { width, height } = element.getBoundingClientRect()
}
```

**React 示例：**
```tsx
// 错误示例：样式更改与布局查询穿插
function Box({ isHighlighted }: { isHighlighted: boolean }) {
  const ref = useRef<HTMLDivElement>(null)
  
  useEffect(() => {
    if (ref.current && isHighlighted) {
      ref.current.style.width = '100px'
      const width = ref.current.offsetWidth // 强制布局
      ref.current.style.height = '200px'
    }
  }, [isHighlighted])
  
  return <div ref={ref}>Content</div>
}

// 正确示例：切换类
function Box({ isHighlighted }: { isHighlighted: boolean }) {
  return (
    <div className={isHighlighted ? 'highlighted-box' : ''}>
      Content
    </div>
  )
}
```

在可能的情况下优先使用 CSS 类而非内联样式。CSS 文件会被浏览器缓存，类提供了更好的关注点分离且更易于维护。

查看[这个 gist](https://gist.github.com/paulirish/5d52fb081b3570c81e3a) 和 [CSS Triggers](https://csstriggers.com/) 了解更多关于强制布局操作的信息。
