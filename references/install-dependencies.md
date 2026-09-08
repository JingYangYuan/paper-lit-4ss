# lit module 安装说明

本发布包用于中英文文献综述、CNKI/Google Scholar 精准补充、本地文献库联动和假设推导。CNKI 检索依赖 ZCode 内置浏览器控制（内置能力，无需安装）；Zotero 与 Zotero MCP 是可选增强，只有当用户希望保存论文、管理全文 PDF、检索本地库或深读已收藏文献时才要求启用。

## 安装模式

| 模式 | 适用场景 | 必需依赖 | Zotero 状态 |
|------|----------|----------|-------------|
| 轻量检索 | 只需要在线检索、摘要筛选和综述写作，不保存全文 | ZCode 内置浏览器控制（内置） | 可跳过，记录为 `用户明确暂缓` |
| 文献库联动 | 需要检查已有文献、去重、导入题录 | ZCode 内置浏览器控制 + Zotero Desktop/Connector | 启用 Zotero |
| 全文保存/深读 | 需要保存 PDF、读取 Zotero 附件全文 | ZCode 内置浏览器控制 + Zotero Desktop/Connector + Zotero MCP | 启用 Zotero MCP |

**选择铁律**：Phase 0/Step 0Q 必须询问用户是否启用 Zotero 和 Zotero MCP。用户不想保存全文或不使用本地库时，不得强制安装 Zotero；把本地文献库阶段记录为 `用户明确暂缓`，继续执行 WebSearch、CNKI、Google Scholar 和摘要核验。

## 强制依赖：ZCode 内置浏览器控制

CNKI 阶段使用 ZCode 内置浏览器控制（browser-use，`mcp__node_repl__js` + browser client），**无需安装任何额外依赖**：ZCode 桌面版自带该能力，浏览器面板对用户可见，验证码和登录由用户在面板中手动完成。

安装后检查（每次 CNKI 阶段开始前执行可用性检查）：

1. browser-use 可列标签页/新建标签页/导航到 `about:blank` 或 `https://kns.cnki.net`，并读取 URL/title 轻量状态。
2. 打开 `https://kns.cnki.net/starter/advanced` 后检查页头机构信息（"大学/学院名 + 手机号"）确认机构授权；未登录时提示用户先在浏览器面板完成机构登录，未登录只能检索题录、不能下载全文。

状态词表：

- `浏览器控制正常`：可列页/新建页/导航且检索页可达。
- `浏览器控制不可用`：工具抛错、无法列页/新建页/导航；停止 CNKI 阶段，提示用户重启宿主会话，不得用 WebSearch/Scholar 替代。
- `浏览器页面未完成` / `CNKI 页面未完成`：页面加载未完成或选择器失配；重试一次后仍失败即停止并记录。

验证码与下载约定：验证码出现时停止自动化并请用户在浏览器面板手动拖动完成；PDF 下载不走浏览器下载管线（必弹 Save-As），使用 `scripts/cnki/kns8s-download.sh`（Cookie + curl）免弹窗下载。完整协议见 [cnki-kns8s-closed-loop.md](cnki-kns8s-closed-loop.md)。

Google Scholar 阶段无需浏览器，用 WebFetch/WebSearch 直接访问即可。

## PDF 归档校验

项目内下载器在归档前必须实际解析 PDF。安装或确认以下任一工具可用：Python 包 pypdf、Poppler 的 pdfinfo，或 qpdf。当前环境缺少三者时，下载器会保留文件为未归档状态，而不会只根据 %PDF 文件头声称成功。

```bash
python3 -c "import pypdf" || command -v pdfinfo || command -v qpdf
```

## 可选增强：Zotero Desktop / Connector

仅在用户选择保存题录、保存全文、本地库联动或 PDF 深读时安装。

官方入口：

- Zotero Desktop: [zotero.org/download](https://www.zotero.org/download/)
- Zotero Connector: [Zotero Connector 文档](https://www.zotero.org/support/connector)
- Better BibTeX: [Better BibTeX for Zotero](https://retorque.re/zotero-better-bibtex/index.html)

安装顺序：

1. 安装 Zotero Desktop。
2. 安装浏览器对应的 Zotero Connector。
3. 打开 Zotero Desktop，并保持运行。
4. 在 Chrome 中确认 Zotero Connector 图标可用。
5. 如需稳定 citation key，安装 Better BibTeX for Zotero。

验收标准：

- Zotero Desktop 能打开本地文献库。
- Zotero Connector 能把当前论文页保存到 Zotero。
- 测试条目包含标题、作者、年份、来源和 URL/DOI 中的多数元数据。
- 如需全文保存，测试条目应能关联 PDF 附件。

没有 Zotero 时，本地文献库阶段不得标记为 `已执行`；应记录为 `用户明确暂缓` 或 `能力缺失`，继续其他在线检索阶段。

## 可选增强：Zotero MCP

Zotero MCP 只在用户需要 Agent 直接搜索 Zotero、读取条目元数据或读取附件全文时启用。本模块不假设宿主已经安装或暴露任何固定名称的 Zotero MCP；安装者应选择自己可用的 Zotero MCP 实现，并按该工具项目的安装说明完成配置。

功能验收：

- 能按关键词检索 Zotero 条目。
- 能读取指定条目的标题、作者、年份、DOI/URL、期刊/出版社等元数据。
- 如用户需要全文深读，能读取指定条目或 PDF 附件全文。

通用配置形态如下，具体 `command`、`args`、工具名称和环境变量以所选 Zotero MCP 项目为准：

```json
{
  "mcpServers": {
    "zotero-mcp": {
      "command": "uvx",
      "args": ["SELECTED_ZOTERO_MCP_PACKAGE_OR_MODULE"],
      "env": {
        "ZOTERO_API_KEY": "YOUR_ZOTERO_API_KEY",
        "ZOTERO_LIBRARY_ID": "YOUR_LIBRARY_ID",
        "ZOTERO_LIBRARY_TYPE": "user"
      }
    }
  }
}
```

本地 Zotero Connector 也可提供轻量保存能力，常见本地接口为：

```text
http://127.0.0.1:23119/connector/saveItems
```

**限制**：Zotero MCP 或 Connector 只解决保存、检索和全文读取；不能替代摘要准入规则。所有正式纳入论文仍必须有摘要或等价全文摘要信息。

## 在 paper-master-4ss 中的位置

本模块已经内置于 ``。依赖检查只用于 CNKI、Google Scholar、Zotero 和本地浏览器能力。

内置文件检查：

```text

├── SKILL.md
├── phases/
├── references/
└── scripts/cnki/
```

## 依赖验收

```bash
test -f SKILL.md
test -f phases/phase-1-search.md
test -f references/cnki-kns8s-closed-loop.md
test -f scripts/cnki/kns8s-download.sh
```

人工确认：

- 浏览器控制可列页/新建页/导航且页面可达。
- CNKI 检索页可在浏览器面板打开；如遇验证码，用户能手动完成。
- Google Scholar 页面完成可达性检查。
- 如用户选择 Zotero：Zotero Desktop/Connector 可保存测试文献。
- 如用户选择 Zotero MCP：所选 Zotero MCP 实现可完成条目检索、元数据读取；需要全文深读时还要能读取附件全文。

未通过浏览器控制可用性检查时，不得正式检索 CNKI。CNKI 阶段不得由 WebSearch、Google Scholar、普通网页搜索或代理替代。未通过 Zotero/Zotero MCP 验收时，只影响本地库和全文保存阶段，不影响在线检索与摘要核验。
