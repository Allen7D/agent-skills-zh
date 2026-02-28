---
title: CSS content-visibility 处理长列表
impact: 高
impactDescription: 更快的初始渲染
tags: rendering, css, content-visibility, long-lists
---

## CSS content-visibility 处理长列表

应用 `content-visibility: auto` 来延迟屏幕外的渲染。

**CSS：**

```css
.message-item {
  content-visibility: auto;
  contain-intrinsic-size: 0 80px;
}
```

**例子：**

```tsx
function MessageList({ messages }: { messages: Message[] }) {
  return (
    <div className="overflow-y-auto h-screen">
      {messages.map(msg => (
        <div key={msg.id} className="message-item">
          <Avatar user={msg.author} />
          <div>{msg.content}</div>
        </div>
      ))}
    </div>
  )
}
```

对于 1000 条消息，浏览器跳过约 990 个屏幕外项目的布局/绘制（初始渲染快 10倍）。
