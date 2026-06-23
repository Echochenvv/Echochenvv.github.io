# Echochenvv.github.io

Echochenvv 的个人博客，使用 Hexo 生成并部署到 GitHub Pages。

- 线上地址：[https://echochenvv.github.io/](https://echochenvv.github.io/)
- 源码分支：`source`
- 发布分支：`master`
- 本地源码目录：`/Users/huangzixiang/Projects/Echochenvv-hexo-blog`
- 本地发布目录：`/Users/huangzixiang/Projects/Echochenvv.github.io-publish`

## 本地开发

```bash
cd /Users/huangzixiang/Projects/Echochenvv-hexo-blog
npm install
npm run server
```

## 构建

```bash
npm run clean
npm run build
```

构建产物会生成到 `public/`，再同步到 `master` 分支用于 GitHub Pages 发布。

## 当前主题

当前版本参考 Anyway.FM 的网页风格做了视觉改版，包括红色页面外框、局部点阵背景、内容流、侧栏模块、黑色栏目标签、dotted leader lines 和移动端红色 top bar。
