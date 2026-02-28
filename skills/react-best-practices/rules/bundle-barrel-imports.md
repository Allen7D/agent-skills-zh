---
title: 避免 Barrel 文件导入
impact: 关键
impactDescription: 200-800ms 导入成本，缓慢的构建
tags: bundle, imports, tree-shaking, barrel-files, performance
---

## 避免 Barrel 文件导入

直接从源文件导入而不是从 barrel 文件导入，以避免加载数千个未使用的模块。**Barrel 文件**是重新导出多个模块的入口点（例如，做 `export * from './module'` 的 `index.js`）。

流行的图标和组件库在其入口文件中可能有**多达 10,000 个重新导出**。对于许多 React 包，**仅导入它们就需要 200-800ms**，影响开发速度和生产环境的冷启动。

**为什么 tree-shaking 不起作用：** 当库被标记为外部（不打包）时，打包器无法优化它。如果您打包它以启用 tree-shaking，分析整个模块图会显著放慢构建。

**错误示例（导入整个库）：**

```tsx
import { Check, X, Menu } from 'lucide-react'
// 加载 1,583 个模块，在开发中额外需要 ~2.8s
// 运行时成本：每次冷启动 200-800ms

import { Button, TextField } from '@mui/material'
// 加载 2,225 个模块，在开发中额外需要 ~4.2s
```

**正确示例（只导入所需的）：**

```tsx
import Check from 'lucide-react/dist/esm/icons/check'
import X from 'lucide-react/dist/esm/icons/x'
import Menu from 'lucide-react/dist/esm/icons/menu'
// 只加载 3 个模块（~2KB vs ~1MB）

import Button from '@mui/material/Button'
import TextField from '@mui/material/TextField'
// 只加载您使用的
```

**替代方案（Next.js 13.5+）：**

```js
// next.config.js - 使用 optimizePackageImports
module.exports = {
  experimental: {
    optimizePackageImports: ['lucide-react', '@mui/material']
  }
}

// 然后您可以保留符合人体工程学的 barrel 导入：
import { Check, X, Menu } from 'lucide-react'
// 在构建时自动转换为直接导入
```

直接导入提供 15-70% 更快的开发启动、28% 更快的构建、40% 更快的冷启动，以及显著更快的 HMR。

常受影响的库：`lucide-react`、`@mui/material`、`@mui/icons-material`、`@tabler/icons-react`、`react-icons`、`@headlessui/react`、`@radix-ui/react-*`、`lodash`、`ramda`、`date-fns`、`rxjs`、`react-use`。

参考：[我们如何在 Next.js 中优化包导入](https://vercel.com/blog/how-we-optimized-package-imports-in-next-js)
