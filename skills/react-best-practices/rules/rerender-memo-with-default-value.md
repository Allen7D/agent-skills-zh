---

title: 将 Memo 组件的默认非原始参数值提取为常量
impact: MEDIUM
impactDescription: 用常量恢复 memo 化
tags: rerender, memo, optimization
---

## 将 Memo 组件的默认非原始参数值提取为常量

当 memo 组件的某个可选参数（如数组、函数或对象）有默认值时，如果调用组件时未传该参数，会导致 memo 失效。因为每次渲染都会创建新实例，`memo()` 的严格相等比较无法命中。

解决方法：将默认值提取为常量。

**错误（`onClick` 每次渲染值都不同）：**

```tsx
const UserAvatar = memo(function UserAvatar({ onClick = () => {} }: { onClick?: () => void }) {
  // ...
})

// 未传 onClick
<UserAvatar />
```

**正确（默认值稳定）：**

```tsx
const NOOP = () => {};

const UserAvatar = memo(function UserAvatar({ onClick = NOOP }: { onClick?: () => void }) {
  // ...
})

// 未传 onClick
<UserAvatar />
```
