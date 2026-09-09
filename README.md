<p align="center">
  <img src="docs/banner.svg" alt="paper-lit-4ss" width="100%">
</p>

# Paper 文献综述 4SS（lit 模块独立版）

中英文双语文献综述与假设推导一体化技能。支持五种模式：完整文献地图（A）、定向综述（B）、快速概览（C）、文献综述+假设推导（D）、知网专项搜索（E）。自动搜索本地文献库、CNKI 中文文献（ZCode 内置浏览器控制 kns8s 专业检索）、Google Scholar、WebSearch、Annual Reviews，生成结构化文献景观地图；收到理论、规范或阐释设计报告时在既有流程中组织支持立场、竞争立场和反例材料，不强制假设推导。当用户需要写文献综述、做系统回顾、找研究空白、提出研究假设、搜索中英文文献时使用。

本包由 `paper-master-4ss/scripts/export_standalone.py` 从总控包 `paper-master-4ss/modules/lit/` 自动导出：

- 包内相对路径相对本包根目录解析；
- `master/` 与 `references/` 中的协议/治理文件是导出时拷贝的快照；
- 跨模块路径 `paper-master-4ss/modules/<x>/...` 相对同级安装的总控包解析；
- 更新方式：修改总控包对应模块后运行
  `python3 paper-master-4ss/scripts/export_standalone.py lit` 重新导出，勿直接编辑本包。

## 它做什么

按固定检索顺序做中英文文献综述：方向确认 → WebSearch → 本地库 / Zotero MCP（可选）→ Annual Reviews → 引文链 → CNKI / Google Scholar 精准补充 → 文献地图。模式 D 才推导假设。

Zotero 是可选增强。启用后按 `references/zotero-local-mcp.md` 做能力检查、本地库检索、摘要即时入库和全文深读；未启用不得强制安装。

## 五种模式

| 模式 | 用途 |
|---|---|
| A 完整地图 | 6 轮以上检索，40–80 篇 |
| B 定向综述 | 4 轮，15–30 篇 |
| C 快速概览 | 2 轮，10–20 篇 |
| D 综述+假设 | 5 轮以上，含假设推导 |
| E 知网专项 | CNKI kns8s 闭环为主 |

## 使用

将本目录安装为宿主 skill（与 `paper-master-4ss` 总控包同级）。入口见 `SKILL.md`。CNKI 依赖可见浏览器控制；Google Scholar 用 WebFetch/WebSearch；Zotero MCP 按 `references/install-dependencies.md` 验收。
