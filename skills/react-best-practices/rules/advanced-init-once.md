---
title: 初始化应用一次，而非每次挂载
impact: 低-中等
impactDescription: 避免开发中的重复初始化
tags: initialization, useEffect, app-startup, side-effects
---

## 初始化应用一次，而非每次挂载

不要将必须在每次应用加载时运行一次的全应用初始化放在组件的 `useEffect([])` 内。组件可能重新挂载，effects 会重新运行。请使用模块级守卫或在入口模块中进行顶级初始化。

**错误示例（在开发环境中运行两次，重新挂载时重新运行）：**

```tsx
function Comp() {
  useEffect(() => {
    loadFromStorage()
    checkAuthToken()
  }, [])

  // ...
}
```

**正确示例（每次应用加载只运行一次）：**

```tsx
let didInit = false

function Comp() {
  useEffect(() => {
    if (didInit) return
    didInit = true
    loadFromStorage()
    checkAuthToken()
  }, [])

  // ...
}
```

参考：[初始化应用](https://react.dev/learn/you-might-not-need-an-effect#initializing-the-application)
