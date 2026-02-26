# YoonaDa Blog

一个基于 Hexo 与 hexo-theme-matery 的个人技术与生活博客，记录后端开发、运维实践、Docker / K8s、大数据、中间件等相关内容，同时也沉淀生活点滴（相册、视频、友链等）。

在线访问地址：<https://blog.yoonada.cn>

## 项目定位

- 个人技术博客与知识笔记仓库
- 使用 Hexo 6 生成静态站点，支持一键构建与部署
- 基于 matery 主题做了中文化与个性化定制（相册、视频、友链等页面）
- 适合作为个人博客参考示例或二次定制使用

## 技术栈

- 静态博客框架：Hexo 6.2.0
- 主题：hexo-theme-matery（位于 `themes/matery`，在原主题基础上做了自定义）
- 模板与样式：EJS、Material Design 风格、响应式布局
- 常用 Hexo 插件：
  - `hexo-generator-*`（分类、标签、归档、首页、搜索等）
  - `hexo-blog-encrypt`（文章加密）
  - `hexo-wordcount`（字数与阅读时长统计）
  - `hexo-permalink-pinyin`（中文标题转拼音链接）
  - `hexo-helper-live2d`、`hexo-lazyload-image` 等增强体验的插件

## 功能特点

- 多端自适应：PC、平板、手机均有良好展示效果
- 分类 / 标签 / 归档页：结构化浏览技术文章
- 文章加密：支持对敏感内容设置访问密码
- 阅读体验：
  - 代码高亮、目录（TOC）、字数与阅读时长统计
  - 复制时可附加版权信息（由主题和插件支持）
- 多媒体展示：
  - 图库：`/gallery` 下划分多个相册（旅行、婚礼等）
  - 视频页：如婚礼视频等
- 社交与互动：
  - 友链页面（`/friends`）
  - 评论与访问统计（由主题集成的评论与统计组件提供）

## 快速开始

### 环境要求

- Node.js（建议 16+）
- Git

### 克隆仓库

```bash
git clone https://github.com/YoonaDa/yoonada-blog.git
cd yoonada-blog
```

### 安装依赖

```bash
npm install
```

### 本地开发预览

```bash
# 启动本地服务，默认端口 4000
npm run server
```

启动后访问：<http://localhost:4000> 即可预览博客。

### 构建与部署

```bash
# 清理旧的构建产物
npm run clean

# 仅生成静态文件（输出到 public/）
npm run build

# 生成并部署（需要在 _config.yml 中配置 deploy）
npm run deploy
```

部署方式由 Hexo 的 `deploy` 配置决定，常见为推送到 GitHub Pages 或自建服务器上的 Git 仓库。

## 目录结构概览

```text
.
├── source/               # 博客源文件（文章、页面、数据等）
│   ├── _posts/           # 所有 Markdown 文章
│   ├── _data/            # 自定义数据（友链、相册、视频等 JSON）
│   ├── about/            # 关于页面
│   ├── gallery/          # 图库与相册页面
│   ├── videos/           # 视频页面
│   ├── friends/          # 友情链接页面
│   └── 404.md            # 自定义 404 页面
├── themes/
│   └── matery/           # 主体使用的 Hexo 主题及其资源
├── scaffolds/            # 文章、页面等的模板
├── _config.yml           # Hexo 全局配置
├── _config.landscape.yml # Hexo 默认主题配置（保留备用）
├── package.json          # npm 脚本与依赖
└── README.md             # 本说明文件
```

## 写作与内容管理

### 新建文章

在项目根目录执行：

```bash
hexo new post "我的新文章"
```

Hexo 会在 `source/_posts` 目录下创建一个同名 Markdown 文件，编辑后重新生成即可。

### 新建页面

例如新建一个名为 `projects` 的页面：

```bash
hexo new page "projects"
```

在生成的 `source/projects/index.md` 中填写 Front-matter 和内容，并在主题或导航配置中添加入口即可。

## 主题与自定义

- 主题目录：`themes/matery`
- 主题配置：主题下的 `_config.yml`
- 更多主题配置、进阶用法请参考：
  - 主题自带文档：[themes/matery/README_CN.md](themes/matery/README_CN.md)
  - 原主题仓库：<https://github.com/blinkfox/hexo-theme-matery>

一般可通过修改：

- 站点基础信息（根目录 `_config.yml`）
- 导航菜单、社交链接、页脚等（主题 `_config.yml` 与 `layout/_partial`）
- 自定义样式（`themes/matery/source/css/my.css` 等）

实现完全符合自己需求的个性化博客。

## 贡献与反馈

- 欢迎通过 Issue 提交 Bug 反馈或改进建议
- 欢迎提交 PR，一起完善配置与示例
- 如有主题或 Hexo 使用问题，也可在 Issue 中讨论

## 版权与许可

- 博客文章与原创内容版权归作者 **YoonaDa** 所有，转载请注明出处
- 主题基于开源项目 **hexo-theme-matery**，请遵循原主题及相关第三方库的开源协议
- 本仓库代码与配置文件仅供学习与参考使用，如需用于生产环境，请根据自身需求调整配置并自查安全性
