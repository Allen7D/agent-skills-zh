---
name: vercel-react-best-practices
description: 来自 Vercel 工程团队的 React 和 Next.js 性能优化指南。编写、评审或重构 React/Next.js 代码时应使用本技能，以确保最佳性能模式。适用于涉及 React 组件、Next.js 页面、数据获取、包体优化或性能提升的任务。
license: MIT
metadata:
  author: vercel
  version: "1.0.0"
---

# Vercel React 最佳实践

由 Vercel 维护的 React 和 Next.js 应用性能优化综合指南。包含 8 个类别下的 57 条规则，按影响力优先级排序，用于指导自动化重构和代码生成。

## 适用场景

在以下情况下参考这些指南：
- 编写新的 React 组件或 Next.js 页面
- 实现数据获取（客户端或服务端）
- 评审代码以发现性能问题
- 重构现有 React/Next.js 代码
- 优化包体积或加载时间

## 规则类别及优先级

| 优先级 | 类别 | 影响 | 前缀 |
|--------|------|------|------|
| 1 | 消除瀑布链 | 致命 | `async-` |
| 2 | 包体积优化 | 致命 | `bundle-` |
| 3 | 服务端性能 | 高 | `server-` |
| 4 | 客户端数据获取 | 中高 | `client-` |
| 5 | 重渲染优化 | 中 | `rerender-` |
| 6 | 渲染性能 | 中 | `rendering-` |
| 7 | JavaScript 性能 | 中低 | `js-` |
| 8 | 高级模式 | 低 | `advanced-` |

## 快速参考

### 1. 消除瀑布链（致命）

- `async-defer-await` - 仅在实际需要时再 await
- `async-parallel` - 独立操作用 Promise.all() 并行
- `async-dependencies` - 部分依赖用 better-all
- `async-api-routes` - API 路由中尽早启动 promise，延后 await
- `async-suspense-boundaries` - 用 Suspense 流式渲染内容

### 2. 包体积优化（致命）

- `bundle-barrel-imports` - 直接导入，避免 barrel 文件
- `bundle-dynamic-imports` - 重组件用 next/dynamic 动态加载
- `bundle-defer-third-party` - 分析/日志等三方库延后到水合后加载
- `bundle-conditional` - 仅在功能激活时加载模块
- `bundle-preload` - 悬停/聚焦时预加载提升感知速度

### 3. 服务端性能（高）

- `server-auth-actions` - 服务端操作如 API 路由需鉴权
- `server-cache-react` - 用 React.cache() 实现每请求去重
- `server-cache-lru` - 跨请求缓存用 LRU
- `server-dedup-props` - 避免 RSC props 重复序列化
- `server-serialization` - 最小化传递给客户端组件的数据
- `server-parallel-fetching` - 组件结构调整以并行获取数据
- `server-after-nonblocking` - 非阻塞操作用 after()

### 4. 客户端数据获取（中高）

- `client-swr-dedup` - 用 SWR 自动去重请求
- `client-event-listeners` - 全局事件监听去重
- `client-passive-event-listeners` - 滚动监听用 passive
- `client-localstorage-schema` - localStorage 数据需版本化并最小化

### 5. 重渲染优化（中）

- `rerender-defer-reads` - 仅在回调中用到的 state 不要订阅
- `rerender-memo` - 耗时操作提取为 memo 组件
- `rerender-memo-with-default-value` - 非原始类型默认 props 上提
- `rerender-dependencies` - effect 依赖用原始类型
- `rerender-derived-state` - 订阅派生布尔值而非原始值
- `rerender-derived-state-no-effect` - 派生状态应在渲染时而非 effect
- `rerender-functional-setstate` - 用函数式 setState 保持回调稳定
- `rerender-lazy-state-init` - 用函数初始化 useState 以延迟计算
- `rerender-simple-expression-in-memo` - 简单原始值无需 memo
- `rerender-move-effect-to-event` - 交互逻辑放到事件处理器
- `rerender-transitions` - 非紧急更新用 startTransition
- `rerender-use-ref-transient-values` - 高频临时值用 ref

### 6. 渲染性能（中）

- `rendering-animate-svg-wrapper` - 动画应包裹 div 而非 SVG 元素
- `rendering-content-visibility` - 长列表用 content-visibility
- `rendering-hoist-jsx` - 静态 JSX 提取到组件外
- `rendering-svg-precision` - 降低 SVG 坐标精度
- `rendering-hydration-no-flicker` - 客户端数据用内联脚本防闪烁
- `rendering-hydration-suppress-warning` - 抑制预期的水合警告
- `rendering-activity` - 显隐用 Activity 组件
- `rendering-conditional-render` - 条件渲染用三元表达式而非 &&
- `rendering-usetransition-loading` - loading 状态优先用 useTransition

### 7. JavaScript 性能（中低）

- `js-batch-dom-css` - 通过类名或 cssText 批量修改 CSS
- `js-index-maps` - 多次查找用 Map
- `js-cache-property-access` - 循环中缓存对象属性
- `js-cache-function-results` - 用模块级 Map 缓存函数结果
- `js-cache-storage` - 缓存 localStorage/sessionStorage 读取
- `js-combine-iterations` - 多次 filter/map 合并为一次循环
- `js-length-check-first` - 先检查数组长度再做昂贵比较
- `js-early-exit` - 函数尽早 return
- `js-hoist-regexp` - RegExp 创建提到循环外
- `js-min-max-loop` - 求 min/max 用循环而非 sort
- `js-set-map-lookups` - 查找用 Set/Map 实现 O(1)
- `js-tosorted-immutable` - 用 toSorted() 保持不可变性

### 8. 高级模式（低）

- `advanced-event-handler-refs` - 事件处理器存到 ref
- `advanced-init-once` - 应用仅初始化一次
- `advanced-use-latest` - 用 useLatest 保持回调引用稳定

## 如何使用

阅读各规则文件以获取详细解释和代码示例：

```
rules/async-parallel.md
rules/bundle-barrel-imports.md
```

每条规则文件包含：
- 简要说明及其重要性
- 错误代码示例及解释
- 正确代码示例及解释
- 额外背景与参考资料

## 完整文档

所有规则展开版请见：`AGENTS.md`
