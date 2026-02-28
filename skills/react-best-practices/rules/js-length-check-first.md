---
title: 数组比较的早期长度检查
impact: 中等-高
impactDescription: 当长度不同时避免昂贵操作
tags: javascript, arrays, performance, optimization, comparison
---

## 数组比较的早期长度检查

当使用昂贵操作（排序、深度相等、序列化）比较数组时，首先检查长度。如果长度不同，数组不可能相等。

在实际应用中，当比较在热点路径（事件处理器、渲染循环）中运行时，这种优化特别有价值。

**错误示例（始终运行昂贵比较）：**

```typescript
function hasChanges(current: string[], original: string[]) {
  // 始终进行排序和连接，即使长度不同
  return current.sort().join() !== original.sort().join()
}
```

即使 `current.length` 是 5，`original.length` 是 100，两个 O(n log n) 排序仍然会运行。还有连接数组和比较字符串的开销。

**正确示例（首先进行 O(1) 长度检查）：**

```typescript
function hasChanges(current: string[], original: string[]) {
  // 如果长度不同则早期返回
  if (current.length !== original.length) {
    return true
  }
  // 只在长度匹配时才排序
  const currentSorted = current.toSorted()
  const originalSorted = original.toSorted()
  for (let i = 0; i < currentSorted.length; i++) {
    if (currentSorted[i] !== originalSorted[i]) {
      return true
    }
  }
  return false
}
```

这种新方法更高效，因为：
- 当长度不同时避免了排序和连接数组的开销
- 避免了连接字符串的内存消耗（对大数组特别重要）
- 避免了破坏原始数组
- 在发现差异时早期返回
