# MeBlog-Vue

基于 Vue 3 开发的现代化博客系统，包含完整的前后台管理功能。

## 项目概述

MeBlog-Vue 是一个功能完备的博客系统，采用 Vue 3 + Element Plus 构建，支持响应式设计，适配各种设备屏幕。系统分为前台展示和后台管理两部分，提供了丰富的博客功能和管理工具。

## 核心功能介绍

### 前台功能
- **文章浏览**：文章列表、文章详情、分类文章列表、标签文章列表
- **内容导航**：分类列表、标签列表、归档列表
- **搜索功能**：支持文章关键词搜索
- **Wiki 系统**：Wiki 列表、Wiki 详情、Wiki 目录导航
- **交互功能**：文章评论、图片放大预览、音乐播放器
- **响应式设计**：适配桌面端、平板端和移动端

### 后台管理
- **文章管理**：文章发布、编辑、删除、分类标签管理
- **分类管理**：分类的增删改查
- **标签管理**：标签的增删改查
- **用户管理**：用户的增删改查、角色分配
- **角色管理**：角色的增删改查、权限分配
- **Wiki 管理**：Wiki 页面的增删改查、目录管理
- **博客设置**：网站基本信息配置
- **数据统计**：文章 PV 统计、发布日历等数据可视化

## 技术栈说明

| 技术/框架 | 版本 | 用途 |
|----------|------|------|
| Vue | 3.5.13 | 前端框架 |
| Element Plus | 2.3.9 | UI 组件库 |
| Vue Router | 4.5.1 | 路由管理 |
| Pinia | 3.0.2 | 状态管理 |
| Axios | 1.9.0 | HTTP 客户端 |
| Vite | 6.2.4 | 构建工具 |
| ECharts | 5.6.0 | 数据可视化 |
| GSAP | 3.13.0 | 动画库 |
| Highlight.js | 11.11.1 | 代码高亮 |
| md-editor-v3 | 5.6.1 | Markdown 编辑器 |
| NProgress | 0.2.0 | 页面加载进度条 |
| Animate.css | 4.1.1 | CSS 动画库 |
| Flowbite | 1.8.1 | Tailwind CSS 组件库 |
| Tailwind CSS | 3.4.17 | CSS 框架 |

## 环境要求

- Node.js >= 14.0.0
- npm >= 6.0.0 或 yarn >= 1.0.0

## 安装与配置步骤

### 1. 克隆仓库

```bash
git clone https://github.com/aipno/meblog-vue.git
cd meblog-vue
```

### 2. 安装依赖

使用 npm：

```bash
npm install
```

或使用 yarn：

```bash
yarn install
```

### 3. 配置环境变量

在项目根目录创建 `.env` 文件，配置必要的环境变量：

```env
# API 基础 URL
VITE_API_BASE_URL=http://localhost:3000/api

# 博客标题
VITE_BLOG_TITLE=MeBlog
```

### 4. 启动开发服务器

使用 npm：

```bash
npm run dev
```

或使用 yarn：

```bash
yarn dev
```

## 使用指南

### 开发模式

启动开发服务器后，访问 `http://localhost:5173` 即可查看前台页面，访问 `http://localhost:5173/admin` 可进入后台管理页面。

### 构建生产版本

使用 npm：

```bash
npm run build
```

或使用 yarn：

```bash
yarn build
```

构建后的文件将输出到 `dist` 目录。

### 预览生产版本

使用 npm：

```bash
npm run preview
```

或使用 yarn：

```bash
yarn preview
```

## 贡献规范

### 代码风格

- 遵循 Vue 官方代码风格指南
- 使用 ES6+ 语法
- 组件命名采用 PascalCase 格式
- 变量和函数命名采用 camelCase 格式

### 提交规范

提交信息应遵循以下格式：

```
<type>(<scope>): <subject>
```

其中：
- `type`：提交类型，如 feat、fix、docs、style、refactor、test、chore 等
- `scope`：修改的范围，如组件名、功能模块等
- `subject`：提交的简短描述

### 分支管理

- `main`：主分支，用于发布生产版本
- `develop`：开发分支，用于集成开发功能
- `feature/xxx`：功能分支，用于开发新功能
- `bugfix/xxx`：修复分支，用于修复 bug

## 许可证信息

本项目采用 AGPL-3.0 许可证。详见 [LICENSE](LICENSE) 文件。

## 联系方式

- 项目地址：https://github.com/aipno/meblog-vue
- 问题反馈：https://github.com/aipno/meblog-vue/issues

## 致谢

感谢所有为本项目做出贡献的开发者和社区！
