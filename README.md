# WChecklist

一个开发中的 Obsidian 插件：通过文件 Properties 筛选清单笔记，在聚合工作台查看和编辑 Markdown 任务，支持 Emoji / 行内属性、任务菜单与详情编辑、属性 Chip 等能力。

本仓库是 WChecklist 的**公开发布仓库**，仅包含插件 Release 产物与说明文档，源代码在私有仓库中开发。

## 安装（公开 Beta，通过 BRAT）

1. 在 Obsidian 社区插件市场安装并启用 [BRAT](https://github.com/TfTHacker/obsidian42-brat)；
2. 在 BRAT 设置中选择「Add Beta plugin」，填入：

   ```text
   https://github.com/DH3650/obsidian-wchecklist-release
   ```

3. 在 Obsidian 社区插件列表中启用 WChecklist，后续更新由 BRAT 提示。

也可以在本仓库 [Releases](../../releases) 页面手动下载 `main.js`、`manifest.json`、`styles.css`，放入 Vault 的 `.obsidian/plugins/obsidian-wchecklist/` 目录。

## 使用说明

- 当前为桌面离线版，主要在 Windows 实测；不保证所有平台或 Obsidian 历史版本均已验证。
- 插件只管理显式注册（带 `wchecklist-type` 属性）的清单笔记，不会自动纳管普通笔记。
- 版本变更见 [CHANGELOG.md](CHANGELOG.md)；各版本的详细说明见对应 Release。

## 反馈

公测期间的问题与建议请通过本仓库 [Issues](../../issues) 提交，请附 Obsidian 版本、插件版本与复现步骤，不要粘贴包含隐私内容的笔记原文。
