---
title: 避免 RSC Props 的重复序列化
impact: LOW
impactDescription: 通过避免重复序列化减少网络负载
tags: server, rsc, serialization, props, client-components
---

## 避免 RSC Props 的重复序列化

**影响：低（通过避免重复序列化减少网络负载）**

RSC→client 的序列化是按对象引用去重的，而不是按值。相同引用 = 只序列化一次；新引用 = 再序列化。请在客户端做变换（`.toSorted()`, `.filter()`, `.map()`），不要在服务端。

**错误（数组被重复序列化）：**

```tsx
// RSC: 发送 6 个字符串（2 个数组 × 3 个元素）
<ClientList usernames={usernames} usernamesOrdered={usernames.toSorted()} />
```

**正确（只发送 3 个字符串）：**

```tsx
// RSC: 只发送一次
<ClientList usernames={usernames} />

// 客户端：在这里变换
'use client'
const sorted = useMemo(() => [...usernames].sort(), [usernames])
```

**嵌套去重行为：**

去重是递归的。影响因数据类型而异：

- `string[]`, `number[]`, `boolean[]`: **高影响** - 数组和所有原始类型都会被完全复制
- `object[]`: **低影响** - 只复制数组结构，嵌套对象按引用去重

```tsx
// string[] - 全部重复
usernames={['a','b']} sorted={usernames.toSorted()} // 发送 4 个字符串

// object[] - 只复制数组结构
users={[{id:1},{id:2}]} sorted={users.toSorted()} // 发送 2 个数组 + 2 个唯一对象（不是 4 个）
```

**会破坏去重的操作（创建新引用）：**

- 数组：`.toSorted()`, `.filter()`, `.map()`, `.slice()`, `[...arr]`
- 对象：`{...obj}`, `Object.assign()`, `structuredClone()`, `JSON.parse(JSON.stringify())`

**更多示例：**

```tsx
// ❌ 错误
<C users={users} active={users.filter(u => u.active)} />
<C product={product} productName={product.name} />

// ✅ 正确
<C users={users} />
<C product={product} />
// 在客户端做过滤/解构
```

**例外：** 当变换很昂贵或客户端不需要原始数据时，可以传递派生数据。
