---
title: 提取为 Memo 组件
impact: MEDIUM
impactDescription: 支持提前返回
tags: rerender, memo, useMemo, optimization
---

## 提取为 Memo 组件

将耗时操作提取为 memo 组件，以便在计算前可以提前返回。

**错误（即使 loading 也会计算 avatar）：**

```tsx
function Profile({ user, loading }: Props) {
  const avatar = useMemo(() => {
    const id = computeAvatarId(user)
    return <Avatar id={id} />
  }, [user])

  if (loading) return <Skeleton />
  return <div>{avatar}</div>
}
```

**正确（loading 时跳过计算）：**

```tsx
const UserAvatar = memo(function UserAvatar({ user }: { user: User }) {
  const id = useMemo(() => computeAvatarId(user), [user])
  return <Avatar id={id} />
})

function Profile({ user, loading }: Props) {
  if (loading) return <Skeleton />
  return (
    <div>
      <UserAvatar user={user} />
    </div>
  )
}
```

**注意：** 如果你的项目启用了 [React Compiler](https://react.dev/learn/react-compiler)，则无需手动使用 `memo()` 和 `useMemo()`，编译器会自动优化重渲染。
