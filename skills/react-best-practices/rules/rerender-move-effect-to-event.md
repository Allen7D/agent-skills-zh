---
title: 将交互逻辑放在事件处理器中
impact: MEDIUM
impactDescription: 避免 effect 重新运行和重复副作用
tags: rerender, useEffect, events, side-effects, dependencies
---

## 将交互逻辑放在事件处理器中

如果副作用是由特定用户操作（提交、点击、拖拽）触发的，请直接在事件处理器中执行。不要将动作建模为 state + effect；这样会导致 effect 在无关变化时重新运行，并可能重复执行动作。

**错误（事件建模为 state + effect）：**

```tsx
function Form() {
  const [submitted, setSubmitted] = useState(false)
  const theme = useContext(ThemeContext)

  useEffect(() => {
    if (submitted) {
      post('/api/register')
      showToast('Registered', theme)
    }
  }, [submitted, theme])

  return <button onClick={() => setSubmitted(true)}>Submit</button>
}
```

**正确（直接在事件处理器中执行）：**

```tsx
function Form() {
  const theme = useContext(ThemeContext)

  function handleSubmit() {
    post('/api/register')
    showToast('Registered', theme)
  }

  return <button onClick={handleSubmit}>Submit</button>
}
```

参考：[是否应将代码移到事件处理器？](https://react.dev/learn/removing-effect-dependencies#should-this-code-move-to-an-event-handler)
