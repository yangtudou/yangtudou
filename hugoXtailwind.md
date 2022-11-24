# 前言
最近在研究 hugo 主题开发，由于我用的 css 框架是 tailwind，就想着能不能集成一下，不用先写好样式，再拉到 hugo theme 里做测试了。

网上查了一下，方法是有的，自己尝试了一下，没什么问题。就在此做一下记录，一些重要的点标注一下。

# 开始操作
## 第一步
1. 新建一个 hugo 站点
```go
{{*/ / /*}}
hugo new site {{ $site_name }}
```

2. 创建一个基本主题框架（**要先进入刚刚创建的站点里 eg: cd {{ $site_name }}**）
```go
{{*/ /$site_name /*}}
hugo new theme {{ $theme_name }}
```

3. 初始化 npm 并生成一个空 `package.json`（**返回站点目录**）
```go
{{*/ / /*}}
npm init -y
```

## 第二步(以下都在站点根目录操作)
1. 添加 npm 所需要的包
```bash
npm install -D tailwindcss prettier prettier-plugin-tailwindcss npm-run-all
```

### 包说明
`tailwindcss` TailwindCSS 框架

`prettier` HTML/CSS/JavaScript 的代码格式化

`prettier-plugin-tailwindcss` TailwindCSS 的代码格式化插件

`npm-run-all` 顺序或并行运行 NPM 脚本


2. 添加如下内容到 `package.json` (**记得替换掉 `{{ $theme_name }}` 部分，下面也会说到这个文件怎么来的**)
```json
{
  ···
  "scripts": {
    "dev:css": "npx tailwindcss -i input.css -o ./themes/{{ $theme_name }}/assets/css/style.css -w",
    "dev:hugo": "hugo server",
    "dev": "run-p dev:*",
    "build:css": "NODE_ENV=production npx tailwindcss -i input.css -o ./themes/{{ $theme_name }}/assets/css/style.css -m",
    "build:hugo": "hugo",
    "build": "run-s build:*"
  }
  ···
}
```

3. 生成 TailwindCSS 配置
```bash
npx tailwindcss init
```

4. 修改 TailwindCSS 配置，如下：(**其中 content 部分的地址为要开发的主题目录下的 layouts**)
```js
/** @type {import('tailwindcss').Config} */
module.exports = {
  content: ['.layouts/**/*.html'],
  theme: {
    extend: {},
  },
  plugins: [],
}
```

5. 新建一个 css 文件，文件名为 `input.css` 内容如下：
```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

6. 配置好站点的配置文件，修改主题为上面新建的主题`{{ $theme_name }}`
```
···
theme = "{{ $thme_name }}"
···
```

## 第三步(以下步骤都在 {{ $theme_name }} 目录下)
1. 在主题根目录下的 `assets` 新建一个名为 `style.css` 的文件，文件可以为空。
```bash
mkdir assets && cd assets && touch style.css
```
2. 修改根目录下的 `index.html`
```
{{ define "main" }}
  <h1 class="text-3xl text-blue-700 font-bold underline">hugo x tailwindcss</h1>
  <p class="bg-sky-600 text-slate-100">这是用 tailwind css 和 hugo 一起渲染出来的，css 文件会自动生成</p> 
{{ end }}
```
3. 修改主题目录 `layouts/partials` 里的 `head.html`，添加如下：
```go
<head>
  ···
  {{ $style := resources.Get "style.css" }}
  <link rel="stylesheet" href="{{ $style.Permalink }}">
  ···
</head>
```

## 第三步(以下都在站点根目录操作)
1. 开发模式
```bash
npm run dev
```

2. 生产模式
```
npm run build
```

参考教程：
- [https://functional.style/hugo/general/tailwind/](https://functional.style/hugo/general/tailwind/)
- [https://www.wimdeblauwe.com/blog/2021/01/18/using-hugo-with-tailwind-css-2/](https://www.wimdeblauwe.com/blog/2021/01/18/using-hugo-with-tailwind-css-2/)
- [https://www.unsungnovelty.org/posts/03/2022/how-to-add-tailwind-css-3-to-a-hugo-website-in-2022/](https://www.unsungnovelty.org/posts/03/2022/how-to-add-tailwind-css-3-to-a-hugo-website-in-2022/)
