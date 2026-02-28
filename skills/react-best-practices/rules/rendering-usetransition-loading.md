---
title: 优先使用 useTransition 处理 loading 状态
impact: LOW
impactDescription: 减少重渲染并提升代码清晰度
tags: rendering, transitions, useTransition, loading, state
---

## 优先使用 useTransition 处理 loading 状态

使用 `useTransition` 替代手动 `useState` 管理 loading 状态。这样可以获得内置的 `isPending` 状态并自动管理 transitions。

**错误（手动管理 loading 状态）：**

```tsx
function SearchResults() {
  const [query, setQuery] = useState('')
  const [results, setResults] = useState([])
  const [isLoading, setIsLoading] = useState(false)

  const handleSearch = async (value: string) => {
    setIsLoading(true)
    setQuery(value)
    const data = await fetchResults(value)
    setResults(data)
    setIsLoading(false)
  }

  return (
    <>
      <input onChange={(e) => handleSearch(e.target.value)} />
      {isLoading && <Spinner />}
      <ResultsList results={results} />
    </>
  )
}
```

**正确（useTransition 内置 pending 状态）：**

```tsx
import { useTransition, useState } from 'react'

function SearchResults() {
  const [query, setQuery] = useState('')
  const [results, setResults] = useState([])
  const [isPending, startTransition] = useTransition()

  const handleSearch = (value: string) => {
    setQuery(value) // 立即更新输入
    
    startTransition(async () => {
      // 获取并更新结果
      const data = await fetchResults(value)
      setResults(data)
    })
  }

  return (
    <>
      <input onChange={(e) => handleSearch(e.target.value)} />
      {isPending && <Spinner />}
      <ResultsList results={results} />
    </>
  )
}
```

**好处：**

- **自动 pending 状态**：无需手动管理 `setIsLoading(true/false)`
- **错误容忍**：即使 transition 抛错 pending 状态也能正确重置
- **更好响应性**：更新期间保持 UI 响应
- **中断处理**：新 transition 会自动取消上一个 pending

参考：[useTransition](https://react.dev/reference/react/useTransition)
