# WChecklist

<img src="assets/icons/brand.svg" alt="WChecklist" width="64" height="64">

一个开发中的 Obsidian 插件：通过文件 Properties 筛选笔记，在聚合页面查看和编辑 Markdown 任务；目标方案包含 emoji / 行内属性、输入辅助菜单，以及 Microsoft Calendar 排期和提醒的双向同步。

- 插件显示名：`WChecklist`，Manifest ID：`obsidian-wchecklist`（与仓库目录名一致）
- 设计规范：[`doc/design/product_preliminary_design.md`](doc/design/product_preliminary_design.md)（V0.5，§28 为当前规范；仅支持 v2，旧注释方案已废弃）
- 开发计划：[`doc/design/development_plan.md`](doc/design/development_plan.md)
- 详细实施计划：[`doc/design/detailed_development_plan.md`](doc/design/detailed_development_plan.md)（任务分解、模块契约、恢复方案与验收追踪）
- 当前进展：G5/G6/G7 已通过，I12 收口、离线 A+B MVP 已完成。工作台、菜单/详情、单文件注册、Chip/Emoji、性能与恢复工具、品牌及使用说明已交付；日历业务仍待 G8 与 I13–I16，详见 [MVP 状态](doc/development/mvp-status.md)。
- 使用与恢复：见 [离线版指南](doc/offline-user-guide.md)，包括安装升级、注册、Chip 设置、完整备份、重建、重装、卸载及已知限制。roadmap 扩展暂不纳入当前 MVP。
- 品牌素材：[稳定资源与 favicon 说明](assets/icons/README.md)，沿用用户原稿；当前没有独立网页，不声称网页 favicon 已启用。
- 离线验收：[G7 通过记录与限制](doc/development/I12-G7-review.md)。确认不代表已发布或所有平台均已实测。
- 版本变更：[CHANGELOG.md](CHANGELOG.md)，从 v0.1.0 起按特性新增、问题修正和优化等分类记录。

## 安装（公开 Beta）

插件处于公开测试阶段，通过 [BRAT](https://github.com/TfTHacker/obsidian42-brat) 安装：

1. 在 Obsidian 社区插件市场安装并启用 BRAT；
2. 在 BRAT 设置中选择「Add Beta plugin」，填入 `https://github.com/DH3650/obsidian-wchecklist-release`；
3. 在 Obsidian 社区插件列表中启用 WChecklist，后续更新由 BRAT 提示。

公开仓库的 Release 均为 Pre-release，附件仅含运行所需的 `main.js`、`manifest.json`、`styles.css`。当前为桌面离线版，主要在 Windows 实测，其他平台与 Obsidian 历史版本未逐一验证。

## 设计原则（摘要）

- **按字段保存唯一事实**：Markdown 保存可读任务语义；Vault 内独立数据文件保存详细排期、提醒和同步记录。完整备份需要两者，索引可重建；单篇 Markdown 复制不含全部日历关联。
- **显式注册**：仅带合法 `wchecklist-type` / `wchecklist-scope` 的文件纳入管理；绝不自动纳管普通笔记。
- **无损编辑**：所有写回基于精确 source span 的最小补丁；有歧义宁可拒绝，不重排用户原文。
- **日历双向、显式、可恢复**：仅同步显式选择的任务及已关联事件的排期/提醒；单执行实例、字段基线与冲突检查，失败可核对恢复。该能力尚未实现。

## 技术栈

| 层 | 选型 |
| --- | --- |
| 宿主 | Obsidian Plugin API（桌面端优先） |
| 语言 | TypeScript（strict） |
| 包管理 | pnpm（packageManager 固定为 12.11.2） |
| 构建 | esbuild → 单文件 `main.js` |
| 测试 | vitest（纯 Node 跑 `src/core/`，不依赖 Obsidian） |
| Markdown 解析 | remark / mdast + 原文 span/BOM 适配（G1 已确认；正式边界在 I02/I03 验证） |
| YAML | yaml 2.9.1，用于受测注册字段的校验与 token 定位；不整体 dump。任务/节点字段适配在 I03/I05 单独验证，日期保持字符串 |
| 编辑器渲染 | CodeMirror 6 Decorations（Live Preview）+ 阅读模式后处理 |
| 日历同步 | 计划使用 Microsoft Graph；当前未接入，授权与提供方能力留 G8 验证 |
| 凭据存储 | 尚未实现，安全存储方案留 S4/G8 验证；不将 token 写入 Vault |
| 任务 ID | Obsidian Block ID；当前宿主按需生成 UUID，保留既有合法 ID |
| Lint | eslint + 自定义规则：`src/core/` 禁止 import `obsidian` |

## 目录结构（规划）

```text
src/
  core/                  # 纯 TypeScript，零 Obsidian 依赖，可在 Node 下全量测试
    model/               # Task / Note / WorkflowNode / ChecklistFile / Schedule
    markdown/            # lexer / outline-parser / metadata-parser / semantics
    registry/            # ChecklistFileRegistry
    index/               # TaskIndex
    diagnostics/
  adapters/
    obsidian/            # Vault / Editor / MetadataCache / SecretStorage 适配
  __tests__/
    fixtures/            # 脱敏回归样例（真实清单不入库，见下文）
doc/
  design/                # 设计规范与开发计划
```

## 开发环境

```bash
pnpm --version           # 12.11.2；本机 Node 26.4.0，CI 使用 Node 22
pnpm install --frozen-lockfile
pnpm typecheck
pnpm lint
pnpm test                # 非 watch 的公开回归；不读取本地私有配置
pnpm fixtures:check      # 校验公开样例的字节哈希、换行和词法计数
pnpm build               # main.js / main.js.map
pnpm dev                 # esbuild watch
pnpm test:private        # I00 仅检查私有文件可读性；T59 语义回归在 I03 引入
pnpm vault:prepare       # 构建后执行，准备 .user/test-vault 独立 Vault
pnpm spike:source        # I01/S1：仅公开用例的只读源位置报告
pnpm spike:host:prepare  # I01/S2-S3：仅向 .user/test-vault 部署专用测试插件及虚构样例
pnpm spike:frontmatter   # I01/S3：公开虚构用例的字段补丁、无损及拒写报告
pnpm inspect:source      # I02：公开用例的注册状态、源树/span/诊断；不读私有配置
pnpm poc:parse           # I03：公开用例的语义/继承/诊断与 POC-01 报告；默认迁移预览
pnpm bench:index         # I04：500 Task 单文件/十项目初测、事件回放和重建一致性报告
pnpm bench:chips         # I12：500 Task/1000 Chip 三组虚构性能基线
pnpm vault:performance   # 固定测试 Vault 的虚构性能样例，只补缺失
pnpm verify:recovery     # I12：OS 临时目录里的离线重装/重建/记录保留演练
```

若使用 Corepack，可由其读取 `packageManager`；没有 Corepack 时直接安装指定版本 pnpm。依赖精确版本及传递依赖记录在 `package.json` / `pnpm-lock.yaml`。`pnpm-workspace.yaml` 仅允许 esbuild 的安装脚本。

独立 Vault：在 Obsidian 中打开 `.user/test-vault`，允许社区插件并启用 WChecklist，检查开发者控制台，然后禁用、重新启用及重启。准备脚本只部署三个插件文件，不自动启用插件或覆盖现有笔记。实际观察须填入 I00 记录，构建成功不等于宿主验收通过。每次重新构建后重跑准备脚本。

### 自动安装到指定 Vault

在 `.user/dev_config.json` 中增加 `obsidianPluginsDir`，填写 Obsidian 插件根目录，例如 `C:/MyVault/.obsidian/plugins`。保留已有 `fixtures` 字段；完整格式见 `config/dev_config.example.json`。Windows 路径可使用 `/`，使用 `\` 时须按 JSON 写成 `\\`。

```bash
pnpm plugin:install      # 构建成功后同步到配置的插件根目录
pnpm plugin:sync         # 仅同步现有产物，不重新构建
```

工具按仓库文件夹名称创建子目录（本仓库为 `obsidian-wchecklist`），只覆盖 `main.js`、`manifest.json`、`styles.css`，保留 `data.json` 及其他本地文件。绝对路径直接使用，相对路径以仓库根目录解析。配置错误、缺少产物时返回非零退出码，开始写入前会检查全部产物；文件系统写入失败时可能存在部分更新，修复权限后可重新同步。

为了符合 [Obsidian 对目录名和插件 ID 一致的要求](https://docs.obsidian.md/Reference/Manifest)，源码 manifest ID 与仓库目录名一致，均为 `obsidian-wchecklist`；安装工具仍会按目标仓库目录名对齐安装副本的 ID。因此这是独立的开发安装，不会沿用旧 `wchecklist` 安装的启用状态或设置。请在 Obsidian 中启用对应插件；更新后禁用并重新启用，或重启 Obsidian。若旧安装也存在，验证时只启用一个。工具不会自动启用插件。

### 同步到生产 Vault 验证

在 `.user/dev_config.json` 的 `obsidianPluginsDirs_ProductiveEnv` 中填写生产 Vault 的插件根目录列表（字符串数组，示例见 `config/dev_config.example.json`）。

```bash
pnpm plugin:install:prod          # 构建成功后同步到全部生产插件目录
pnpm plugin:sync:prod --dry-run   # 检查现有产物与全部目标，仅预览，不写入
pnpm plugin:sync:prod             # 仅同步现有产物
```

生产命令只使用该列表，不回退到测试目录；写入前检查全部产物和目标类型。仅覆盖各目录下 `obsidian-wchecklist/` 内的 `main.js`、`manifest.json`、`styles.css`，保留 `data.json`、其他插件文件及笔记。生产副本的 manifest 与仓库产物保持一致（本地源码版本为 `0.0.0`，不会自动注入 Release tag）；要求插件 ID 与仓库目录名一致。更新后禁用并重新启用插件，或重启 Obsidian。同步多个目标时，实际磁盘写入失败可能造成部分目录已更新，排除故障后可重跑。

### 私有 fixture（真实清单回归）

真实项目清单含敏感信息，**不入库、不进 CI、不进日志**。本地配置方式：

1. 复制 `config/dev_config.example.json` 为 `.user/dev_config.json`（已被 gitignore）；
2. `fixtures` 为文件路径数组，可指向 `.user/fixtures/` 的副本，或用户明确指定的绝对路径。相对路径以仓库根目录解析；加载器只读源文件，不复制或改写；
3. 默认配置不存在时明确跳过；配置损坏、字段缺失或文件不可读时失败。可用 `WCHECKLIST_PRIVATE_CONFIG` 指定其他配置路径，显式指定却不存在也视为失败。错误信息不打印私有路径或正文。

I00 的四个公开样例均为人工构造；其中全量样例为 40 checkbox（32 Task + 8 Note），不能代替真实 399/398/1 验收。样例清单见 `src/__tests__/fixtures/inventory.json`。其中边界样例的 `lexicalMarkers` 包含代码示例中的伪任务，不能作为解析任务数量。UTF-16 字符长度与 UTF-8 字节长度分别记录，Git 禁止转换 fixture 换行。

I01 另新增 `production-shape.md`：只读参考生产笔记，保留 Tab 层级、空白、标题和 399 个 marker 分布；正文、frontmatter 属性和链接值全部替换为虚构内容。它提供结构回归，不代表完整真实语义验收。公开测试及 `spike:source` 不读取私有配置；后者只接受公开 fixture 目录内的 Markdown，报告限于 `doc/development/reports/`。

S2/S3 的实际宿主步骤见 [I01 操作清单](doc/development/I01-host-checklist.md)。专用插件名为 **WChecklist I01 Spike**，只能在本仓库的独立 Vault 中运行，不进入正式插件构建，也不使用生产插件安装配置。重复准备不会覆盖已经编辑的测试笔记。

I02 默认报告为 `doc/development/reports/I02-source.json`。查看完整源树可执行 `pnpm inspect:source --input src/__tests__/fixtures/boundaries-crlf.md --report doc/development/reports/I02-detail.json --full`；只接受公开 fixture，报告限于公开报告目录。源标记数量不是业务 Task/Note 数量，完整字段契约及后续复验见 [I02 记录](doc/development/iterations/I02.md)。

I03 新增虚构 Tab 语义样例 `I03-semantics.md`，公开 fixture 共六份。`pnpm poc:parse` 默认只读检查公开材料；`--mode registered` 仅输出 active 文件的实体，迁移预览不进入正常索引；`--full` 可用于公开完整报告。显式私有命令 `pnpm poc:parse --private --baseline .user/I03-baseline.json` 使用本地配置，报告仅为 `.user/reports/I03-private.json` 的匿名摘要，禁止完整正文输出和写入公开报告目录。

真实工作笔记会继续变化：历史设计快照为 399/398/1；用户于 2026-10-23 确认新增待办，本轮固定快照为 **400 checkbox / 399 Task / 1 Note**。本地 baseline 绑定 SHA-256 与预期计数，内容变化时重新确认或使用隔离副本，不自动追随源文件改预期；公开结构样例仍保留 399。详见 [I03 记录](doc/development/iterations/I03.md)。

I04 的 `TaskIndex` 提供只读查询、独立 TaskGroup 和版本检查；默认统计排除过期缓存，显式 `includeStale` 可查看旧数据。Waiting 包含 delegated，Note 不进入未完成数；跨文件 Node ID 冲突撤下继承并保留诊断。`pnpm bench:index` 默认用完全虚构的两组 500 Task＋10 Note 数据，报告为 `doc/development/reports/I04-index.json`；`--input` 可附加公开 fixture，`--report` 可指定公开报告路径。Node 初测不代替 I12 的宿主 UI 验收；API、版本规则与证据见 [I04 记录](doc/development/iterations/I04.md)。

## 开发阶段

| 阶段 | 内容 |
| --- | --- |
| Spike | 解析器无损性、HTML 注释渲染、processFrontMatter、Entra 授权、Graph 时区 |
| 批次 A | 离线核心：Registry → Parser（POC-01）→ Index → Command/Mutation（POC-02） |
| 批次 B | Obsidian 集成：工作台 → 迁移向导（L0–L2）→ Chip；完成后为第一个可用版本（无日历） |
| 批次 C | Schedule 编辑 + Graph 单向同步（L3）；第二个版本 |

详见开发计划文档。
