---
title: localStorage 数据的版本化和最小化
impact: 中等
impactDescription: 防止模式冲突，减少存储大小
tags: client, localStorage, storage, versioning, data-minimization
---

## localStorage 数据的版本化和最小化

在键中添加版本前缀，只存储所需字段。防止模式冲突和意外存储敏感数据。

**错误示例：**

```typescript
// 没有版本，存储一切，没有错误处理
localStorage.setItem('userConfig', JSON.stringify(fullUserObject))
const data = localStorage.getItem('userConfig')
```

**正确示例：**

```typescript
const VERSION = 'v2'

function saveConfig(config: { theme: string; language: string }) {
  try {
    localStorage.setItem(`userConfig:${VERSION}`, JSON.stringify(config))
  } catch {
    // 在隐身/私人浏览、配额超限或禁用时抛出异常
  }
}

function loadConfig() {
  try {
    const data = localStorage.getItem(`userConfig:${VERSION}`)
    return data ? JSON.parse(data) : null
  } catch {
    return null
  }
}

// 从 v1 到 v2 的迁移
function migrate() {
  try {
    const v1 = localStorage.getItem('userConfig:v1')
    if (v1) {
      const old = JSON.parse(v1)
      saveConfig({ theme: old.darkMode ? 'dark' : 'light', language: old.lang })
      localStorage.removeItem('userConfig:v1')
    }
  } catch {}
}
```

**从服务器响应中存储最少字段：**

```typescript
// 用户对象有20+个字段，只存储 UI 所需的
function cachePrefs(user: FullUser) {
  try {
    localStorage.setItem('prefs:v1', JSON.stringify({
      theme: user.preferences.theme,
      notifications: user.preferences.notifications
    }))
  } catch {}
}
```

**始终用 try-catch 包装：** 在隐身/私人浏览（Safari、Firefox）、配额超限或禁用时，`getItem()` 和 `setItem()` 会抛出异常。

**好处：** 通过版本化实现模式演化，减少存储大小，防止存储 token/PII/内部标志。
