# React 最佳实践

一个结构化仓库，用于创建和维护针对智能体和大语言模型优化的 React 最佳实践。

## 结构

- `rules/` - 单独规则文件（每条规则一个文件）
  - `_sections.md` - 区块元数据（标题、影响、描述）
  - `_template.md` - 新规则创建模板
  - `area-description.md` - 单独规则文件
- `src/` - 构建脚本和工具
- `metadata.json` - 文档元数据（版本、组织、摘要）
- __`AGENTS.md`__ - 编译输出（自动生成）
- __`test-cases.json`__ - LLM 评估用测试用例（自动生成）

## 快速开始

1. 安装依赖：
   ```bash
   pnpm install
   ```

2. 从规则文件构建 AGENTS.md：
   ```bash
   pnpm build
   ```

3. 校验规则文件：
   ```bash
   pnpm validate
   ```

4. 提取测试用例：
   ```bash
   pnpm extract-tests
   ```

## 创建新规则

1. 复制 `rules/_template.md` 到 `rules/area-description.md`
2. 选择合适的区块前缀：
   - `async-` 消除瀑布链（第1区块）
   - `bundle-` 包体积优化（第2区块）
   - `server-` 服务端性能（第3区块）
   - `client-` 客户端数据获取（第4区块）
   - `rerender-` 重渲染优化（第5区块）
   - `rendering-` 渲染性能（第6区块）
   - `js-` JavaScript 性能（第7区块）
   - `advanced-` 高级模式（第8区块）
3. 填写 frontmatter 和内容
4. 确保有清晰的示例和解释
5. 运行 `pnpm build` 重新生成 AGENTS.md 和 test-cases.json

## 规则文件结构

每个规则文件应遵循如下结构：

```markdown
---
title: 规则标题
impact: MEDIUM
impactDescription: 可选描述
tags: 标签1, 标签2, 标签3
---

## 规则标题

简要说明规则及其重要性。

**错误示例（说明问题）：**

```typescript
// 错误代码示例
```

**正确示例（说明正确做法）：**

```typescript
// 正确代码示例
```

示例后的可选解释文本。

参考：[链接](https://example.com)

## 文件命名规范

- 以 `_` 开头的文件为特殊文件（构建时排除）
- 规则文件：`area-description.md`（如 `async-parallel.md`）
- 区块由文件名前缀自动推断
- 规则按标题字母顺序在每个区块内排序
- ID（如 1.1, 1.2）在构建时自动生成

## 影响等级

- `CRITICAL` - 最高优先级，重大性能提升
- `HIGH` - 显著性能提升
- `MEDIUM-HIGH` - 中高提升
- `MEDIUM` - 中等性能提升
- `LOW-MEDIUM` - 中低提升
- `LOW` - 增量提升

## 脚本

- `pnpm build` - 编译规则到 AGENTS.md
- `pnpm validate` - 校验所有规则文件
- `pnpm extract-tests` - 提取 LLM 评估用测试用例
- `pnpm dev` - 构建并校验

## 贡献说明

添加或修改规则时：

1. 使用正确的区块前缀
2. 遵循 `_template.md` 结构
3. 包含清晰的错误/正确示例及解释
4. 添加合适标签
5. 运行 `pnpm build` 重新生成 AGENTS.md 和 test-cases.json
6. 规则自动按标题排序，无需手动编号！

## 鸣谢

最初由 [@shuding](https://x.com/shuding) 在 [Vercel](https://vercel.com) 创建。
