---
title: 推迟非关键的第三方库
impact: 中等
impactDescription: 在水合后加载
tags: bundle, third-party, analytics, defer
---

## 推迟非关键的第三方库

分析、日志记录和错误跟踪不会阻塞用户交互。在水合后加载它们。

**错误示例（阻塞初始打包）：**

```tsx
import { Analytics } from '@vercel/analytics/react'

export default function RootLayout({ children }) {
  return (
    <html>
      <body>
        {children}
        <Analytics />
      </body>
    </html>
  )
}
```

**正确示例（在水合后加载）：**

```tsx
import dynamic from 'next/dynamic'

const Analytics = dynamic(
  () => import('@vercel/analytics/react').then(m => m.Analytics),
  { ssr: false }
)

export default function RootLayout({ children }) {
  return (
    <html>
      <body>
        {children}
        <Analytics />
      </body>
    </html>
  )
}
```
