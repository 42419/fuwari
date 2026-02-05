---
title: Fuwari 博客，精美的静态博客
published: 2026-02-02
description: 详细介绍如何从零开始部署 Fuwari 精美静态博客，并展示其强大的 Markdown 扩展语法
tags: [Fuwari, Astro, 部署, 博客]
image: "./home.png"
category: 教程
draft: false
---

Fuwari 是一个基于 Astro 构建的精美静态博客模板，拥有流畅的动画、响应式设计和丰富的功能。本文将详细介绍如何从零开始部署你的 Fuwari 博客，并展示其强大的 Markdown 扩展语法。

## 📋 前置要求

在开始之前，请确保你的系统已安装以下工具：

- **Node.js**：版本 >= 20
- **pnpm**：版本 >= 9（推荐使用 pnpm 作为包管理器）

### 安装 pnpm

如果你还没有安装 pnpm，可以运行以下命令：

```bash
npm install -g pnpm
```

## 🚀 第一步：创建项目

有多种方式可以创建 Fuwari 博客项目：

### 方式一：使用 GitHub 模板

1. 访问 [Fuwari GitHub 仓库](https://github.com/saicaca/fuwari)
2. 点击 "Use this template" → "Generate a new repository"
3. 为你的新仓库命名并创建

### 方式二：Fork 仓库

1. 访问 [Fuwari GitHub 仓库](https://github.com/saicaca/fuwari)
2. 点击右上角的 "Fork" 按钮

### 方式三：使用命令行工具

```bash
npm create fuwari@latest
# 或者
pnpm create fuwari@latest
# 或者
yarn create fuwari
# 或者
bun create fuwari@latest
```

## 📥 第二步：克隆项目并安装依赖

创建仓库后，将项目克隆到本地：

```bash
git clone https://github.com/你的用户名/你的仓库名.git
cd 你的仓库名
```

安装项目依赖：

```bash
pnpm install
pnpm add sharp
```

> **注意**：`sharp` 是一个图像处理库，用于图片优化，建议安装以提高性能。

## ⚙️ 第三步：配置博客

编辑 `src/config.ts` 文件来个性化你的博客：

```typescript
export const siteConfig: SiteConfig = {
  title: "你的博客标题",
  subtitle: "你的博客副标题",
  lang: "zh_CN",
  themeColor: {
    hue: 50,
    fixed: false,
  },
  // ... 其他配置项
};
```

在 `astro.config.mjs` 中配置你的网站 URL：

```javascript
export default defineConfig({
  site: "https://你的域名.com",
  base: "/",
  trailingSlash: "always",
  // ... 其他配置
});
```

## ✍️ 第四步：创建第一篇文章

使用以下命令创建新文章：

```bash
pnpm new-post 我的-first-文章
```

这将在 `src/content/posts/` 目录下创建一个新的 Markdown 文件。编辑文件内容，文章的 frontmatter 如下：

```yaml
---
title: 我的 First 文章
published: 2026-02-03
description: 这是我的第一篇文章
image: ./cover.jpg # 可选的封面图
tags: [标签1, 标签2]
category: 分类
draft: false # 设置为 true 可以保存为草稿
---
```

## 🧪 第五步：本地预览

在部署之前，你可以先在本地预览博客效果：

```bash
pnpm dev
```

访问 `http://localhost:4321` 查看你的博客。

构建生产版本：

```bash
pnpm build
```

预览构建后的网站：

```bash
pnpm preview
```

## 🌐 第六步：部署到平台

### Vercel 部署（推荐）

1. 将代码推送到 GitHub
2. 访问 [Vercel](https://vercel.com) 并登录
3. 点击 "Add New Project"
4. 导入你的 GitHub 仓库
5. Vercel 会自动检测 Astro 项目，点击 "Deploy"
6. 等待部署完成，即可获得你的博客链接

### Netlify 部署

1. 将代码推送到 GitHub
2. 访问 [Netlify](https://netlify.com) 并登录
3. 点击 "Add new site" → "Import an existing project"
4. 选择你的 GitHub 仓库
5. 构建命令设置为：`pnpm build`
6. 发布目录设置为：`dist`
7. 点击 "Deploy site"

### GitHub Pages 部署

1. 创建 `.github/workflows/deploy.yml` 文件：

```yaml
name: Deploy to GitHub Pages

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: "npm"
      - run: npm install -g pnpm
      - run: pnpm install
      - run: pnpm build
      - uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./dist
```

1. 在 GitHub 仓库设置中，将 Pages 源设置为 `gh-pages` 分支
2. 推送代码，GitHub Actions 会自动部署

### Cloudflare Pages 部署

1. 将代码推送到 GitHub
2. 访问 [Cloudflare Pages](https://pages.cloudflare.com)
3. 点击 "Create a project" → "Connect to Git"
4. 选择你的 GitHub 仓库
5. 构建命令：`pnpm build`
6. 输出目录：`dist`
7. 点击 "Save and Deploy"

### Cloudflare Workers 部署

如果你选择了 Cloudflare Workers 部署，项目已经包含了 `wrangler.toml` 配置文件：

```toml
name = "fuwari"
compatibility_date = "2026-02-03"

[assets]
directory = "./dist"
```

部署步骤：

```bash
# 构建项目
pnpm build

# 部署到 Cloudflare Workers
npx wrangler pages deploy dist
```

## ✨ Fuwari 核心特性

### 🎨 精美的 UI 设计

- 流畅的页面过渡动画
- 精心设计的配色方案
- 完全响应式布局，支持移动端
- 支持 **明暗模式** 切换

### 📑 自动目录生成

文章会根据标题自动生成目录（TOC），支持滚动高亮和点击跳转。

### 🔍 内置搜索功能

基于 Pagefind 的全文搜索功能，无需配置即可使用。

### 📡 RSS 订阅支持

自动生成 RSS 订阅源，方便读者订阅。

### 🎯 评论功能

支持集成多种评论系统。

### 💬 丰富的 Markdown 扩展语法

Fuwari 支持多种 Markdown 扩展语法，让文章更加丰富多彩。

## 📝 Fuwari Markdown 扩展语法详解

### 1. GitHub 仓库卡片

你可以添加链接到 GitHub 仓库的动态卡片，仓库信息从 GitHub API 实时获取。

::github{repo="saicaca/fuwari"}

**使用方法：**

```markdown
::github{repo="<owner>/<repo>"}
```

**示例：**

```markdown
::github{repo="saicaca/fuwari"}
::github{repo="vercel/next.js"}
```

### 2. 警告框（Admonitions）

支持五种类型的警告框，用于突出显示重要信息。

#### Note - 注意

:::note
突出显示用户应该注意的信息，即使是在快速浏览时。
:::

#### Tip - 提示

:::tip
帮助用户更成功的可选信息。
:::

#### Important - 重要

:::important
用户成功所必需的关键信息。
:::

#### Warning - 警告

:::warning
由于潜在风险而需要立即注意的关键内容。
:::

#### Caution - 谨慎

:::caution
操作的负面潜在后果。
:::

**基本语法：**

```markdown
:::note
注意内容
:::

:::tip
提示内容
:::
```

**自定义标题：**

:::note[自定义标题]
这是一个带有自定义标题的注释。
:::

```markdown
:::note[自定义标题]
这是一个带有自定义标题的注释。
:::
```

**GitHub 语法支持：**

> [!NOTE]
> 也支持 GitHub 风格的语法。

```
> [!NOTE]
> 这是 GitHub 风格的注释。

> [!TIP]
> 这是 GitHub 风格的提示。
```

### 3. 剧透（Spoiler）

你可以在文本中添加剧透内容，鼠标悬停时显示。

内容 :spoiler[是隐藏的 **剧透内容**]！

```markdown
内容 :spoiler[是隐藏的 **剧透内容**]！
```

### 4. Expressive Code 代码块

Fuwari 使用 Expressive Code 提供强大的代码块功能，支持多种高亮和标记方式。

#### 语法高亮

Expressive Code 会自动根据语言标识符进行语法高亮。

```js
console.log("这段代码是语法高亮的！");
```

**使用方法：**

````markdown
```js
console.log("代码");
```
````

#### 代码块标题

使用 `title` 属性添加标题，或在代码第一行添加注释。

```js title="example.js"
function hello() {
  console.log("Hello, World!");
}
```

```html
<!-- index.html -->
<div>文件名注释示例</div>
```

**使用方法：**

````markdown
```js title="文件名.js"
代码内容;
```

```html
<!-- 路径/文件名 -->
<div>内容</div>
```
````

#### 终端框架

自动检测 shell 类型（bash、powershell、sh 等），以终端风格展示命令。

```bash
echo "终端示例"
```

```powershell title="PowerShell 示例"
Write-Output "这个有标题！"
```

**使用方法：**

````markdown
```bash
命令
```

```powershell title="终端标题"
命令
```
````

#### 行高亮标记

在语言标识符后添加花括号，指定行号或范围进行高亮。

```js {1, 3-5}
// 第 1 行会被高亮
// 第 2 行普通
// 第 3-5 行会被高亮
console.log("第 3 行");
console.log("第 4 行");
console.log("第 5 行");
```

**使用方法：**

````markdown
```js {1, 3-5}
代码;
```
````

#### 行标记类型（ins、del）

使用 `del` 和 `ins` 属性标记删除和插入的行。

```js del={2} ins={3-4}
function demo() {
  console.log("这行被标记为已删除");
  console.log("这行被标记为已插入");
  console.log("这行被标记为已插入");
  console.log("普通行");
}
```

**使用方法：**

````markdown
```js del={2} ins={3-4}
代码;
```
````

#### 行内文本标记

在语言标识符后添加引号包裹要标记的文本。

```js "特定文本"
function demo() {
  console.log("标记特定文本");
}
```

**使用方法：**

````markdown
```js "要标记的文本"
代码;
```
````

#### 正则表达式标记

使用斜杠包裹正则表达式来匹配和标记文本。

```ts /ye[sp]/
console.log("单词 yes 和 yep 将被标记。");
```

**使用方法：**

````markdown
```ts /正则表达式/
代码;
```
````

#### 类似 diff 的语法

使用 `diff` 语言标识符，`+` 表示插入，`-` 表示删除。

```diff
+这一行将被标记为已插入
-这一行将被标记为已删除
这是一常规行
```

**使用方法：**

````markdown
```diff
+新增行
-删除行
普通行
```
````

#### 自动换行

使用 `wrap` 属性启用或禁用自动换行。

```js wrap
// 启用自动换行
function getLongString() {
  return "这是一个非常长的字符串，会自动换行";
}
```

**使用方法：**

````markdown
```js wrap
代码;
```

```js wrap=false
代码;
```
````

#### 可折叠部分

使用 `collapse` 属性指定要折叠的行范围，点击可展开/收起。

```js collapse={1-3, 7-9}
// 这部分代码会被折叠
import { boilerplate } from "example";

// 这部分可见
console.log("可见代码");

// 这部分也会被折叠
boilerplate.cleanup();
```

**使用方法：**

````markdown
```js collapse={1-3, 7-9}
代码;
```
````

#### 行号

使用 `showLineNumbers` 启用行号，`startLineNumber` 设置起始行号。

```js showLineNumbers
// 显示行号
console.log("第 2 行");
console.log("第 3 行");
```

```js showLineNumbers startLineNumber=10
// 从第 10 行开始编号
console.log("第 11 行");
```

**使用方法：**

````markdown
```js showLineNumbers
代码;
```

```js showLineNumbers startLineNumber=10
代码;
```
````

### 5. 数学公式

支持 LaTeX 数学公式，包括行内公式和块级公式。

行内公式：$E = mc^2$

块级公式：

$$
\int_0^\infty e^{-x^2} dx = \frac{\sqrt{\pi}}{2}
$$

$$
\begin{aligned}
f(x) &= x^2 + 2x + 1 \\
&= (x + 1)^2
\end{aligned}
$$

**使用方法：**

```markdown
行内公式：$E = mc^2$

块级公式：

$$
\int_0^\infty e^{-x^2} dx = \frac{\sqrt{\pi}}{2}
$$
```

### 6. 图片查看器

点击图片会自动打开 Lightbox 查看器，支持缩放和切换。

![示例图片](https://images.unsplash.com/photo-1518770660439-4636190af475?w=800)

**使用方法：** 使用标准 Markdown 图片语法即可，点击查看器会自动启用。

```markdown
![图片描述](图片URL)
```

## 📝 常用命令

| 命令                       | 说明           |
| -------------------------- | -------------- |
| `pnpm install`             | 安装依赖       |
| `pnpm dev`                 | 启动开发服务器 |
| `pnpm build`               | 构建生产版本   |
| `pnpm preview`             | 预览构建结果   |
| `pnpm new-post <filename>` | 创建新文章     |
| `pnpm format`              | 格式化代码     |
| `pnpm lint`                | 检查代码规范   |

## 🎯 总结

通过以上步骤，你已经成功部署了自己的 Fuwari 博客！Fuwari 提供了丰富的功能和精美的设计，非常适合个人博客、技术文档等内容网站。

### Fuwari 的主要优势

1. **零配置搜索**：基于 Pagefind 的静态搜索，无需后端
2. **丰富的 Markdown 扩展**：支持警告框、代码高亮、数学公式等
3. **优秀的性能**：Astro 构建的静态站点，加载速度快
4. **现代化设计**：流畅的动画和精美的 UI
5. **易于定制**：清晰的配置文件和主题系统

如果你在部署过程中遇到任何问题，可以参考 [Fuwari 官方文档](https://github.com/saicaca/fuwari) 或在 GitHub 上提交 Issue。

祝你博客写作愉快！🎉
