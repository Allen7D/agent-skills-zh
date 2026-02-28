---
title: 为重复查找构建索引映射
impact: 低-中等
impactDescription: 1M 操作减少到 2K 操作
tags: javascript, map, indexing, optimization, performance
---

## 为重复查找构建索引映射

多个通过相同键的 `.find()` 调用应该使用 Map。

**错误示例（每次查找 O(n)）：**

```typescript
function processOrders(orders: Order[], users: User[]) {
  return orders.map(order => ({
    ...order,
    user: users.find(u => u.id === order.userId)
  }))
}
```

**正确示例（每次查找 O(1)）：**

```typescript
function processOrders(orders: Order[], users: User[]) {
  const userById = new Map(users.map(u => [u.id, u]))

  return orders.map(order => ({
    ...order,
    user: userById.get(order.userId)
  }))
}
```

构建 map 一次（O(n)），然后所有查找都是 O(1)。
对于 1000 个订单 × 1000 个用户：1M 操作 → 2K 操作。
