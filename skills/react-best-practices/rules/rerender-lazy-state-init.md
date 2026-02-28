---
title: 使用懒初始化 useState
impact: MEDIUM
impactDescription: 每次渲染都浪费计算
tags: react, hooks, useState, performance, initialization
---

## 使用懒初始化 useState

对于昂贵的初始值，给 `useState` 传递一个函数。若不用函数形式，初始化器会在每次渲染时都运行，即使这个值只用一次。

**错误（每次渲染都运行）：**

```tsx
function FilteredList({ items }: { items: Item[] }) {
  // buildSearchIndex() 每次渲染都运行，即使初始化后
  const [searchIndex, setSearchIndex] = useState(buildSearchIndex(items))
  const [query, setQuery] = useState('')
  
  // 当 query 变化时，buildSearchIndex 也会不必要地再次运行
  return <SearchResults index={searchIndex} query={query} />
}

function UserProfile() {
  // JSON.parse 每次渲染都运行
  const [settings, setSettings] = useState(
    JSON.parse(localStorage.getItem('settings') || '{}')
  )
  
  return <SettingsForm settings={settings} onChange={setSettings} />
}
```

**正确（只运行一次）：**

```tsx
function FilteredList({ items }: { items: Item[] }) {
  // buildSearchIndex() 只在初始渲染时运行
  const [searchIndex, setSearchIndex] = useState(() => buildSearchIndex(items))
  const [query, setQuery] = useState('')
  
  return <SearchResults index={searchIndex} query={query} />
}

function UserProfile() {
  // JSON.parse 只在初始渲染时运行
  const [settings, setSettings] = useState(() => {
    const stored = localStorage.getItem('settings')
    return stored ? JSON.parse(stored) : {}
  })
  
  return <SettingsForm settings={settings} onChange={setSettings} />
}
```

当需要从 localStorage/sessionStorage 计算初始值、构建数据结构（索引、映射）、读取 DOM 或做重计算时，使用懒初始化。

对于简单原始值（`useState(0)`）、直接引用（`useState(props.value)`）或便宜的字面量（`useState({})`），函数形式不是必须的。
