---
title: 基于用户意图的预加载
impact: 中等
impactDescription: 减少感知延迟
tags: bundle, preload, user-intent, hover
---

## 基于用户意图的预加载

在需要之前预加载重型打包以减少感知延迟。

**例子（在悬停/获得焦点时预加载）：**

```tsx
function EditorButton({ onClick }: { onClick: () => void }) {
  const preload = () => {
    if (typeof window !== 'undefined') {
      void import('./monaco-editor')
    }
  }

  return (
    <button
      onMouseEnter={preload}
      onFocus={preload}
      onClick={onClick}
    >
      Open Editor
    </button>
  )
}
```

**例子（在功能开关启用时预加载）：**

```tsx
function FlagsProvider({ children, flags }: Props) {
  useEffect(() => {
    if (flags.editorEnabled && typeof window !== 'undefined') {
      void import('./monaco-editor').then(mod => mod.init())
    }
  }, [flags.editorEnabled])

  return <FlagsContext.Provider value={flags}>
    {children}
  </FlagsContext.Provider>
}
```

`typeof window !== 'undefined'` 检查防止为 SSR 打包预加载的模块，优化服务器打包大小和构建速度。
