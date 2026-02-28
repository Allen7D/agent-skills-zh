---
title: 优化 SVG 精度
impact: 低
impactDescription: 减少文件大小
tags: rendering, svg, optimization, svgo
---

## 优化 SVG 精度

减少 SVG 坐标精度以减少文件大小。最佳精度取决于 viewBox 大小，但一般来说应该考虑减少精度。

**错误示例（过度精度）：**

```svg
<path d="M 10.293847 20.847362 L 30.938472 40.192837" />
```

**正确示例（1 位小数）：**

```svg
<path d="M 10.3 20.8 L 30.9 40.2" />
```

**使用 SVGO 自动化：**

```bash
npx svgo --precision=1 --multipass icon.svg
```
