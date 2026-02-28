---
title: 使用函数式 setState 更新
impact: MEDIUM
impactDescription: 防止闭包失效和不必要的回调重建
tags: react, hooks, useState, useCallback, callbacks, closures
---

## 使用函数式 setState 更新

当基于当前 state 值更新 state 时，使用 setState 的函数式写法，而不是直接引用 state 变量。这样可以防止闭包失效，消除不必要的依赖，并让回调引用保持稳定。

**错误（需要 state 作为依赖）：**

```tsx
function TodoList() {
  const [items, setItems] = useState(initialItems)
  
  // 回调必须依赖 items，每次 items 变化都会重建
  const addItems = useCallback((newItems: Item[]) => {
    setItems([...items, ...newItems])
  }, [items])  // ❌ items 依赖导致重建
  
  // 忘记依赖会有闭包失效风险
  const removeItem = useCallback((id: string) => {
    setItems(items.filter(item => item.id !== id))
  }, [])  // ❌ 缺少 items 依赖 - 会用到旧 items!
  
  return <ItemsEditor items={items} onAdd={addItems} onRemove={removeItem} />
}
```

第一个回调每次 `items` 变化都会重建，可能导致子组件不必要的重渲染。第二个回调有闭包失效 bug——它总是引用初始的 `items`。

**正确（回调稳定，无闭包失效）：**

```tsx
function TodoList() {
  const [items, setItems] = useState(initialItems)
  
  // 稳定回调，永不重建
  const addItems = useCallback((newItems: Item[]) => {
    setItems(curr => [...curr, ...newItems])
  }, [])  // ✅ 无需依赖
  
  // 总是用最新 state，无闭包失效风险
  const removeItem = useCallback((id: string) => {
    setItems(curr => curr.filter(item => item.id !== id))
  }, [])  // ✅ 安全且稳定
  
  return <ItemsEditor items={items} onAdd={addItems} onRemove={removeItem} />
}
```

**好处：**

1. **回调引用稳定** - state 变化时无需重建回调
2. **无闭包失效** - 总是操作最新 state
3. **更少依赖** - 简化依赖数组，减少内存泄漏
4. **防止 bug** - 消除 React 闭包 bug 的常见根源

**何时用函数式更新：**

- 任何 setState 依赖当前 state
- 在 useCallback/useMemo 里需要 state
- 事件处理器引用 state
- 异步操作更新 state

**何时直接赋值没问题：**

- 直接设置静态值：`setCount(0)`
- 只用 props/参数赋值：`setName(newName)`
- 不依赖前一个 state

**注意：** 如果你的项目启用了 [React Compiler](https://react.dev/learn/react-compiler)，编译器可自动优化部分场景，但函数式写法仍推荐用于防止闭包 bug。
