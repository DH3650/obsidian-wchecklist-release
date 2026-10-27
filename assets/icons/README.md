# WChecklist 品牌分发资源

本目录是用户原始设计的稳定分发路径，设计源保留在 `doc/design/WChecklist-Icon-Kit/`，不在此重新绘制或转换图片。

- `brand.svg`、`brand-light.svg`、`brand-mono.svg`、`brand-symbol.svg`：品牌图形。
- `ribbon.svg`、`file-register.svg`、`file-registered.svg`、`diagnostics.svg`、`diagnostics-alert.svg`：宿主 UI 原稿及状态变体。
- `favicon.svg`、`favicon.ico`、`favicon-16x16.png`、`favicon-32x32.png`：网页图标资源。16px PNG 原稿对应源包 `png/favicon/favicon-16.png`，其他文件与源包同名。

`pnpm brand:generate` 原样同步 13 个文件；`pnpm brand:check` 校验字节一致，不覆盖源稿。先修改原稿再同步。CI 与发布门禁检查分发副本，源与副本不能各自修改。

Obsidian 运行时使用从原稿编译生成的 `src/adapters/obsidian/icons.ts`，无需读本目录；插件安装包仍只包含 main.js/manifest.json/styles.css。README 和其他文档可以链接这里的资源。

目前没有独立 HTML 网站，网页 favicon 的部署/浏览器实测记为不适用。以后有网站时按实际部署根路径挂载：

```html
<link rel="icon" href="./assets/icons/favicon.ico" sizes="any">
<link rel="icon" href="./assets/icons/favicon.svg" type="image/svg+xml">
<link rel="icon" href="./assets/icons/favicon-32x32.png" type="image/png" sizes="32x32">
```

相对路径需随真实页面位置调整；资源入库不代表网页已部署。GitHub.com 单个仓库的标签 favicon 由 GitHub 控制，本目录和 README 不会替换它。本轮没有创建网站、修改远端仓库设置或上传 Social Preview。
