---
title: 条件性模块加载
impact: 高
impactDescription: 只在需要时加载大型数据
tags: bundle, conditional-loading, lazy-loading
---

## 条件性模块加载

只在功能被激活时才加载大型数据或模块。

**例子（延迟加载动画帧）：**

```tsx
function AnimationPlayer({ enabled, setEnabled }: { enabled: boolean; setEnabled: React.Dispatch<React.SetStateAction<boolean>> }) {
  const [frames, setFrames] = useState<Frame[] | null>(null)

  useEffect(() => {
    if (enabled && !frames && typeof window !== 'undefined') {
      import('./animation-frames.js')
        .then(mod => setFrames(mod.frames))
        .catch(() => setEnabled(false))
    }
  }, [enabled, frames, setEnabled])

  if (!frames) return <Skeleton />
  return <Canvas frames={frames} />
}
```

`typeof window !== 'undefined'` 检查防止为 SSR 打包此模块，优化服务器打包大小和构建速度。
