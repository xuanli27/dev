# 前端实验室 (QianDu Lab)

![Node.js >= 20](https://img.shields.io/badge/node.js-%3E%3D20-brightgreen)
![pnpm >= 9](https://img.shields.io/badge/pnpm-%3E%3D9-blue)

基于 [Astro](https://astro.build) 构建的现代化博客网站，专注于前端技术研究与分享。

[**🖥️ 在线访问**](https://qiandulab.com)

![Preview Image](https://raw.githubusercontent.com/saicaca/resource/main/fuwari/home.png)

## 关于

前端实验室（QianDu Lab）是一个专注于前端技术研究与分享的技术博客，致力于探索 Web 开发的最佳实践，分享实用的开发经验和技术见解。

## ✨ Features

- [x] 基于 [Astro](https://astro.build) 和 [Tailwind CSS](https://tailwindcss.com) 构建
- [x] 流畅的动画和页面过渡效果
- [x] 亮色/暗色主题切换
- [x] 可自定义的主题色和横幅
- [x] 响应式设计
- [x] 集成 [Pagefind](https://pagefind.app/) 搜索功能
- [x] 扩展的 Markdown 语法支持
- [x] 文章目录
- [x] RSS 订阅

## 🚀 快速开始

1. 克隆仓库：
   ```sh
   git clone https://github.com/qiandulab/qiandulab-blog.git
   cd qiandulab-blog
   ```

2. 安装依赖：
   ```sh
   pnpm install
   ```
   如果未安装 [pnpm](https://pnpm.io)，请先运行 `npm install -g pnpm`

3. 本地开发：
   ```sh
   pnpm dev
   ```

4. 构建生产版本：
   ```sh
   pnpm build
   ```

## 📝 文章配置

```yaml
---
title: 我的第一篇博客
published: 2023-09-09
description: 这是我的第一篇技术博客文章
image: ./cover.jpg
tags: [前端, JavaScript]
category: 前端开发
draft: false
lang: zh_CN
---
```

## 🧩 扩展的 Markdown 语法

除了 Astro 默认支持的 [GitHub Flavored Markdown](https://github.github.com/gfm/) 外，还包含以下扩展功能：

- 提示框（Admonitions）
- GitHub 仓库卡片
- 增强的代码块（基于 [Expressive Code](https://expressive-code.com/)）

## ⚡ 命令说明

所有命令都应在项目根目录的终端中运行：

| 命令                       | 说明                                    |
|:---------------------------|:---------------------------------------|
| `pnpm install`             | 安装依赖                                |
| `pnpm dev`                 | 启动本地开发服务器（`localhost:4321`）    |
| `pnpm build`               | 构建生产版本到 `./dist/`                |
| `pnpm preview`             | 本地预览构建结果                         |
| `pnpm check`               | 检查代码错误                            |
| `pnpm format`              | 使用 Biome 格式化代码                   |
| `pnpm new-post <filename>` | 创建新文章                              |
| `pnpm astro ...`           | 运行 Astro CLI 命令                     |
| `pnpm astro --help`        | 获取 Astro CLI 帮助                     |

## 📄 许可证

本项目基于 MIT 许可证开源。

## 🙏 致谢

本项目基于 [Fuwari](https://github.com/saicaca/fuwari) 模板构建，感谢原作者的优秀工作。
