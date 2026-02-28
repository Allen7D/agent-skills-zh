---
title: 使用循环而非排序来找最大/最小值
impact: 低
impactDescription: O(n) 而非 O(n log n)
tags: javascript, arrays, performance, sorting, algorithms
---

## 使用循环而非排序来找最大/最小值

找到最小或最大元素只需要单次遍历数组。排序是浪费的且更慢。

**错误示例（O(n log n) - 排序来找最新的）：**

```typescript
interface Project {
  id: string
  name: string
  updatedAt: number
}

function getLatestProject(projects: Project[]) {
  const sorted = [...projects].sort((a, b) => b.updatedAt - a.updatedAt)
  return sorted[0]
}
```

为了找最大值而对整个数组进行排序。

**错误示例（O(n log n) - 为最旧和最新进行排序）：**

```typescript
function getOldestAndNewest(projects: Project[]) {
  const sorted = [...projects].sort((a, b) => a.updatedAt - b.updatedAt)
  return { oldest: sorted[0], newest: sorted[sorted.length - 1] }
}
```

当只需要最小/最大值时仍然不必要地进行排序。

**正确示例（O(n) - 单次循环）：**

```typescript
function getLatestProject(projects: Project[]) {
  if (projects.length === 0) return null
  
  let latest = projects[0]
  
  for (let i = 1; i < projects.length; i++) {
    if (projects[i].updatedAt > latest.updatedAt) {
      latest = projects[i]
    }
  }
  
  return latest
}

function getOldestAndNewest(projects: Project[]) {
  if (projects.length === 0) return { oldest: null, newest: null }
  
  let oldest = projects[0]
  let newest = projects[0]
  
  for (let i = 1; i < projects.length; i++) {
    if (projects[i].updatedAt < oldest.updatedAt) oldest = projects[i]
    if (projects[i].updatedAt > newest.updatedAt) newest = projects[i]
  }
  
  return { oldest, newest }
}
```

单次遍历数组，无复制，无排序。

**替代方案（对小数组使用 Math.min/Math.max）：**

```typescript
const numbers = [5, 2, 8, 1, 9]
const min = Math.min(...numbers)
const max = Math.max(...numbers)
```

这对小数组有效，但由于扩展运算符的限制，对于非常大的数组可能会更慢或直接抛出错误。在 Chrome 143 中最大数组长度约为 124000，在 Safari 18 中约为 638000；确切数字可能会有所不同 - 参见[这个示例](https://jsfiddle.net/qw1jabsx/4/)。为了可靠性，请使用循环方法。
