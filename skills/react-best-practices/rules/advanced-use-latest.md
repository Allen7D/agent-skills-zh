---
title: 使用 useEffectEvent 获得稳定的回调引用
impact: 低
impactDescription: 防止 effect 重新运行
tags: advanced, hooks, useEffectEvent, refs, optimization
---

## 使用 useEffectEvent 获得稳定的回调引用

在回调中访问最新值而无需将它们添加到依赖数组中。防止 effect 重新运行，同时避免过时的闭包。

**错误示例（每次回调变化时 effect 都会重新运行）：**

```tsx
function SearchInput({ onSearch }: { onSearch: (q: string) => void }) {
  const [query, setQuery] = useState('')

  useEffect(() => {
    const timeout = setTimeout(() => onSearch(query), 300)
    return () => clearTimeout(timeout)
  }, [query, onSearch])
}
```

**正确示例（使用 React 的 useEffectEvent）：**

```tsx
import { useEffectEvent } from 'react';

function SearchInput({ onSearch }: { onSearch: (q: string) => void }) {
  const [query, setQuery] = useState('')
  const onSearchEvent = useEffectEvent(onSearch)

  useEffect(() => {
    const timeout = setTimeout(() => onSearchEvent(query), 300)
    return () => clearTimeout(timeout)
  }, [query])
}
```
