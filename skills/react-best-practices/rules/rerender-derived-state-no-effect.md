---
title: 渲染时计算派生状态
impact: MEDIUM
impactDescription: 避免冗余渲染和状态漂移
tags: rerender, derived-state, useEffect, state
---

## 渲染时计算派生状态

如果某个值可以从当前 props/state 计算出来，不要存到 state 或用 effect 更新。直接在渲染时派生，避免多余渲染和状态漂移。不要仅因 prop 变化就在 effect 里 setState；优先用派生值或 key 重置。

**错误（冗余 state 和 effect）：**

```tsx
function Form() {
  const [firstName, setFirstName] = useState('First')
  const [lastName, setLastName] = useState('Last')
  const [fullName, setFullName] = useState('')

  useEffect(() => {
    setFullName(firstName + ' ' + lastName)
  }, [firstName, lastName])

  return <p>{fullName}</p>
}
```

**正确（渲染时派生）：**

```tsx
function Form() {
  const [firstName, setFirstName] = useState('First')
  const [lastName, setLastName] = useState('Last')
  const fullName = firstName + ' ' + lastName

  return <p>{fullName}</p>
}
```

参考：[你可能不需要 effect](https://react.dev/learn/you-might-not-need-an-effect)
