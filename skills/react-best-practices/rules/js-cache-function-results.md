---
title: 缓存重复的函数调用
impact: 中等
impactDescription: 避免冗余计算
tags: javascript, cache, memoization, performance
---

## 缓存重复的函数调用

在渲染过程中，当相同的函数使用相同的输入被重复调用时，使用模块级别的 Map 来缓存函数结果。

**错误示例（冗余计算）：**

```typescript
function ProjectList({ projects }: { projects: Project[] }) {
  return (
    <div>
      {projects.map(project => {
        // 对相同的项目名称调用 slugify() 100+ 次
        const slug = slugify(project.name)
        
        return <ProjectCard key={project.id} slug={slug} />
      })}
    </div>
  )
}
```

**正确示例（缓存结果）：**

```typescript
// 模块级别缓存
const slugifyCache = new Map<string, string>()

function cachedSlugify(text: string): string {
  if (slugifyCache.has(text)) {
    return slugifyCache.get(text)!
  }
  const result = slugify(text)
  slugifyCache.set(text, result)
  return result
}

function ProjectList({ projects }: { projects: Project[] }) {
  return (
    <div>
      {projects.map(project => {
        // 每个唯一项目名称只计算一次
        const slug = cachedSlugify(project.name)
        
        return <ProjectCard key={project.id} slug={slug} />
      })}
    </div>
  )
}
```

**单值函数的更简单模式：**

```typescript
let isLoggedInCache: boolean | null = null

function isLoggedIn(): boolean {
  if (isLoggedInCache !== null) {
    return isLoggedInCache
  }
  
  isLoggedInCache = document.cookie.includes('auth=')
  return isLoggedInCache
}

// 认证变化时清除缓存
function onAuthChange() {
  isLoggedInCache = null
}
```

使用 Map（而不是 hook），这样它可以在任何地方工作：工具函数、事件处理器，而不仅仅是 React 组件。

参考：[我们如何让 Vercel 仪表板快两倍](https://vercel.com/blog/how-we-made-the-vercel-dashboard-twice-as-fast)
