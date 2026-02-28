---
title: 在 Refs 中存储事件处理器
impact: 低
impactDescription: 稳定的订阅
tags: advanced, hooks, refs, event-handlers, optimization
---

## 在 Refs 中存储事件处理器

当在不应该因回调变化而重新订阅的 effects 中使用回调时，将回调存储在 refs 中。

**错误示例（每次渲染都重新订阅）：**

```tsx
function useWindowEvent(event: string, handler: (e) => void) {
  useEffect(() => {
    window.addEventListener(event, handler)
    return () => window.removeEventListener(event, handler)
  }, [event, handler])
}
```

**正确示例（稳定的订阅）：**

```tsx
function useWindowEvent(event: string, handler: (e) => void) {
  const handlerRef = useRef(handler)
  useEffect(() => {
    handlerRef.current = handler
  }, [handler])

  useEffect(() => {
    const listener = (e) => handlerRef.current(e)
    window.addEventListener(event, listener)
    return () => window.removeEventListener(event, listener)
  }, [event])
}
```

**替代方案：如果您使用最新版本的 React，可以使用 `useEffectEvent`：**

```tsx
import { useEffectEvent } from 'react'

function useWindowEvent(event: string, handler: (e) => void) {
  const onEvent = useEffectEvent(handler)

  useEffect(() => {
    window.addEventListener(event, onEvent)
    return () => window.removeEventListener(event, onEvent)
  }, [event])
}
```

`useEffectEvent` 为相同模式提供了更简洁的 API：它创建一个稳定的函数引用，总是调用处理器的最新版本。
