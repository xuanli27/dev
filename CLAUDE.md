# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

Fuwari - 基于 Astro 的静态博客模板，使用 Tailwind CSS + Svelte 组件。

**技术栈**: Astro 5.x | Tailwind CSS | Svelte 5.x | pnpm 9.x | Biome | Pagefind

## 常用命令

```bash
pnpm install              # 安装依赖
pnpm dev                  # 开发服务器 (localhost:4321)
pnpm build                # 构建 + 生成搜索索引
pnpm preview              # 预览构建
pnpm check                # 类型检查
pnpm format               # 格式化代码
pnpm lint                 # 检查并修复
pnpm new-post <filename>  # 创建新文章
```

## 核心架构

### 关键文件

- `src/config.ts` - 站点配置（信息、导航、主题色等）
- `src/content/config.ts` - 内容集合 schema 定义
- `astro.config.mjs` - Astro 配置（部署前需修改 site/base）

### 目录结构

```
src/
├── content/          # 内容集合
│   ├── posts/       # 博客文章 (Markdown)
│   └── spec/        # 规范文档
├── components/      # Astro + Svelte 组件
│   ├── widget/     # 侧边栏小部件
│   ├── control/    # 控制组件
│   └── misc/       # 其他组件
├── plugins/         # 自定义插件
│   ├── expressive-code/  # 代码块插件
│   ├── remark-*.js      # Markdown 处理
│   └── rehype-*.mjs     # HTML 处理
├── layouts/         # 页面布局
├── pages/           # 路由页面
├── utils/           # 工具函数
├── i18n/            # 国际化
└── styles/          # 全局样式
```

### Markdown 处理流程

**Remark** (Markdown AST) → **Rehype** (HTML AST)

关键插件：
- `remarkReadingTime` - 阅读时间
- `remarkExcerpt` - 摘要提取
- `remarkGithubAdmonitionsToDirectives` - 提示框转换
- `rehypeComponents` - 自定义组件渲染（GitHub 卡片、提示框）
- `rehypeKatex` - 数学公式

### 主题系统

- CSS 变量控制（`--primary`、`--codeblock-bg` 等）
- 亮/暗模式切换：`LightDarkSwitch.svelte`
- 主题色调配置：`src/config.ts` 中 hue 值 (0-360)

### 页面转换

Swup 实现平滑动画，配置在 `astro.config.mjs`。

## 开发规范

### 代码风格

- Tab 缩进 + 双引号
- 提交前：`pnpm check && pnpm format`
- 遵循 Conventional Commits

### 部署配置

修改 `astro.config.mjs`:
```js
export default defineConfig({
  site: "https://your-domain.com/",
  base: "/",  // 子路径部署时修改
});
```

### 创建文章

```bash
pnpm new-post <filename>
```

Frontmatter 字段：
- `title` (必需)、`published` (必需)
- `description`、`tags`、`category`、`draft` (可选)

### Markdown 扩展

- GitHub 提示框 (Note/Tip/Important/Caution/Warning)
- GitHub 仓库卡片
- 数学公式 (KaTeX)
- 增强代码块 (Expressive Code)

### 添加图标

```bash
pnpm add @iconify-json/<icon-set-name>
```
图标查找：https://icones.js.org/

### 搜索

Pagefind 在构建时自动生成索引，必须使用 `pnpm build`（不是 `astro build`）。
