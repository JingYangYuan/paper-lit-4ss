# Phase 1：文献搜索

由 `lit module/SKILL.md` 在所有模式下加载。

## 顾问派发闸门

搜索开始前必须读取 `search-strategy-consultant` 的意见并写入 `agent-synthesis-lit-[YYYY-MM-DD].md`。候选论文清单形成后，派发 `agents/screening-consultant.md` 复核纳入/排除/待核验分类；进入下一 phase 前，把筛选结论追加到 `paper-workspace/_logs/agents/lit-[YYYY-MM-DD]/agent-synthesis-lit-[YYYY-MM-DD].md`。

## 搜索顺序（固定，不可调换）

```
Step 0a: 检索方向预确认             → ask_user 确认检索方向并映射中英文检索词
Step 0Q: 检索阶段预确认             → ask_user 确认各阶段执行/暂缓
Step 1: WebSearch 先行             → 识别关键词变体 + 核心文献 + 摘要总结
Step 1Q: ask_user            → 确认关键词、噪音、下一步方向
Step 2: 知识图谱（如有）            → 补充预提取的发现和关系
Step 2Q: ask_user            → 确认理论、机制或人群方向
Step 3: 本地文献库（如有）          → 匹配已收集文献，通读 PDF
Step 3Q: ask_user            → 确认已有文献缺口
Step 4: Annual Reviews              → 综述文章检查点
Step 4Q: ask_user            → 确认经典脉络和综述方向
Step 5: 引文链扩展                  → 向前/向后追踪
Step 5Q: ask_user            → 确认最终补洞方向
Step 6: CNKI/Scholar 可达性检查     → 两者都检查或记录用户明确暂缓
Step 7: 浏览器控制最终精准补充     → 可达且未暂缓的 CNKI/Scholar 精准补缺口
```

**核心逻辑**：先询问用户本次检索阶段是否全部启用；再用 WebSearch 摸清领域关键词和核心文献面貌；每完成一个搜索阶段，先让用户确认方向，避免无效堆积；再做本地匹配、综述检查和引文链；最后根据已确认的缺口，用 CNKI 和 Google Scholar 做精准补充。CNKI 和 Google Scholar 不再用于中段宽泛搜索，但不得被默认跳过。

## Step 0a：检索方向预确认（强制第一步）

任何检索、目录创建、日志初始化或顾问检索建议落地前，必须先 ask_user 确认本次检索方向。用户选择后，把每个方向映射为具体中英文检索词对，写入 `SEARCH_LOG` 的 `检索方向映射` 表。

```text
question: "请确认本次文献检索的方向与范围（可多选辅助确认）"
header: "检索方向"
options: [
  {label: "X→Y 主效应", description: "搜索核心自变量对因变量的直接影响"},
  {label: "X 概念族", description: "搜索核心自变量的测量、成因、趋势和相邻概念"},
  {label: "Y 概念族", description: "搜索因变量的概念谱系、测量和相邻结果"},
  {label: "M 机制变量", description: "搜索机制、中介、过程和解释链条"},
  {label: "W 调节变量", description: "搜索制度、区域、人群、行业、时期等调节条件"},
  {label: "IV 工具变量", description: "搜索工具变量、自然实验、政策冲击或识别策略先例"},
  {label: "竞品核查", description: "专门搜索与本研究问题高度相近的已有研究"}
]
```

未选中的方向标记为 `用户暂不覆盖`。后续检索若偶然命中这些方向，可以记录在候选清单中，但不得主动扩展为新的检索轮次，除非用户在阶段确认中重新选择。

## Step 0Q：检索阶段预确认（强制）

任何检索开始前，必须先调用 ask_user，询问本次是否启用以下阶段：

1. WebSearch 先行探路
2. 本地文献库 / 已有 PDF
3. Zotero / Zotero MCP（保存题录、保存全文、读取附件全文时启用）
4. Annual Reviews 综述检查点
5. 引文链扩展
6. CNKI 中文文献
7. Google Scholar 英文文献

默认建议启用所有与模式相关的在线检索阶段。CNKI 与 Google Scholar 不得默认跳过；除非用户明确选择 `暂缓`，否则必须至少完成可达性检查，并在 `SEARCH_LOG` 记录为以下状态之一：

```
已执行 / CNKI 已执行 / 用户明确暂缓 / 网络不可达 / 浏览器控制正常 / 浏览器控制不可用 / 浏览器页面未完成 / CNKI 页面未完成
```

### 开场问询模板

结构化示例模块统一遵守 `references/ask-user-question-examples.md`。0Q 可使用以下 ask_user 示例：

```text
question: "本次文献检索阶段如何安排？CNKI 和 Google Scholar 不得默认跳过，Zotero 可按是否保存全文决定。"
header: "检索阶段"
options: [
  {label: "在线全启用", description: "启用 WebSearch、Annual Reviews、引文链、CNKI 和 Google Scholar，Zotero 暂缓"},
  {label: "全部启用", description: "同时启用本地文献库和 Zotero/Zotero MCP，适合保存题录、全文或读取附件"},
  {label: "指定阶段", description: "用户逐项指定启用或暂缓的来源；CNKI/Scholar 只能由用户明确暂缓"}
]
```

```
本次文献检索将分阶段进行，避免无效堆积。请确认启用哪些阶段：
- WebSearch：先摸清关键词、核心文献和摘要
- 本地文献库：检查已有文献/PDF；不使用本地库时可暂缓
- Zotero/Zotero MCP：保存题录/全文、读取 Zotero 附件全文；不保存全文时可暂缓
- Annual Reviews：检查经典综述脉络
- 引文链扩展：追踪经典和近年前沿
- CNKI：后段精准补充中文文献，不默认跳过
- Google Scholar：后段精准补充英文文献，不默认跳过

问题：本次检索阶段如何安排？
选项A：在线检索全启用，Zotero 暂缓（推荐给不保存全文的用户）
选项B：全部启用，包括 Zotero/Zotero MCP
选项C：按用户指定阶段执行
```

用户若选择 C，必须追问或从用户文本中提取每个阶段的状态。CNKI 和 Google Scholar 只能被标记为 `用户明确暂缓`，不能因为模式、环境猜测或时间节省而静默跳过。Zotero/Zotero MCP 可以由用户明确暂缓；暂缓只影响本地库、全文保存和附件全文读取，不影响在线检索与摘要核验。

## 阶段确认闸门（强制）

每个搜索阶段结束后，必须先写入搜索日志和阶段综述段落，然后调用 ask_user。不得在未获得用户确认时继续下一阶段。

### 问询内容模板

各阶段确认可复用以下 ask_user 示例：

```text
question: "本阶段已完成。下一阶段按哪个方向继续？"
header: "方向确认"
options: [
  {label: "按推荐继续", description: "采用当前搜索日志中证据最充分的关键词、理论或机制方向"},
  {label: "收窄方向", description: "收窄到用户指定的理论、机制、人群、时期或中文/英文文献缺口"},
  {label: "改查方向", description: "放弃当前推荐方向，改用用户指定的新检索方向"}
]
```

```
本阶段新增：
- 有摘要且可纳入：N篇（列3-5篇代表论文）
- 无摘要/仅题录：N篇（暂不纳入）
- 疑似噪音：N篇（说明原因）

当前判断：
- 已清晰的关键词/理论/机制：
- 仍缺的方向：
- 建议下一阶段检索式：

问题：下一阶段按哪个方向继续？
选项A：按推荐方向继续
选项B：收窄到[某理论/机制/人群/时期]
选项C：改查[用户指定方向]
```

### 用户确认后的处理

- 用户选 A：按推荐检索式进入下一阶段。
- 用户选 B：立即改写下一阶段检索式，并把收窄理由写入日志。
- 用户选 C：使用用户指定方向，旧方向只保留为备选。
- 用户要求停止：保存当前日志和论文清单，终止后续搜索。

### 阶段综述沉淀

每个检索阶段完成后，必须输出并写入一段 200-400 字中文综述段落，综合该阶段新增论文的核心发现、方法特征、证据强弱和与本研究问题的关系。所有阶段段落追加到：

```text
paper-workspace/02-literature/stage-syntheses.md
```

阶段综述是 Phase 4 文献综述草稿的素材，不得只用论文罗列表替代。若某阶段无可纳入论文，也要写明无新增证据的原因、待核验缺口和下一阶段补洞方向。

## 摘要准入规则（强制）

所有进入正式论文清单、Phase 2 文献地图、Phase 3 假设推导或 Phase 4 草稿的论文，必须具备以下之一：

- 数据库/期刊页面可抓取摘要；
- PDF/全文中可提取摘要、引言摘要段或等价的研究概述；
- 综述/书籍章节无标准摘要时，必须提取至少150字的核心论点摘要。

标题、作者、期刊、引用数、下载数和搜索结果片段只能用于候选排序，不能用于正式纳入。无摘要、仅题录、只有标题作者的文献，一律放入"待核验/排除候选"，不得作为证据引用。每篇正式纳入论文在清单中必须填写：

```
摘要状态：已抓取摘要 / 全文可替代摘要
摘要要点：研究问题 + 方法/材料 + 核心发现
```

**执行约束**：每篇候选至少抓取摘要、数据库详情页摘要、PDF/全文摘要段或等价全文概述后，才允许判断为保留。不得只看标题决定纳入，不得用"看起来相关"替代摘要核验。

**Zotero 摘要存储约束**：每篇进入正式论文清单且相关度为 H 或 M 的论文，必须在抓取摘要后立即尝试存入 Zotero，保存内容必须包含 `abstractNote` 或等价摘要字段。当前宿主若暴露 Zotero MCP，则优先使用 Zotero MCP；若只有 Zotero Connector 或本地导出能力，则记录连接器保存状态；若 Zotero 不可用，写入 `paper-workspace/02-literature/abstracts-pending-zotero.md`，并在搜索日志中记录 `Zotero 不可用，摘要未保存`。不得等检索结束后再批量补存摘要。

**浏览器控制可用性硬约束**：进入任何 CNKI 检索动作前，必须先完成 Step 6.0 的浏览器控制可用性检查：ZCode 内置浏览器控制（browser-use）可列标签页/新建标签页/导航，能打开 `about:blank` 或 CNKI 首页并读取轻量页面状态。检查未通过时，不得进入 CNKI 检索页、不得执行任何 CNKI 页面脚本、不得写 `CNKI 已执行`。browser-use 不可用时记录 `浏览器控制不可用`，停止 CNKI 阶段并提示用户重启宿主会话；不得 kill 进程、不得用 WebSearch、Google Scholar、普通搜索、`cnki-researcher` 或 lit agents 替代 CNKI 检索结果。CNKI 需要登录、验证码或人工确认时，在用户可见的浏览器面板中完成。

---

## Step 1：WebSearch 先行探路（所有模式）

**目的**：快速识别领域核心关键词（含同义词/中英对照）、找到 5-10 篇核心文献、对摘要进行初步总结。为后续搜索提供方向校准。

### 1a. 关键词勘探

运行 2-3 个宽泛检索式，摸清领域术语面貌：

```
"[主要概念]" "[结果变量]" sociology OR demography
"[主要概念]" review OR meta-analysis OR 综述
"[主要概念]" "[中文对应词]"
```

**产出**：关键词变体清单（含中英文对照、同义词、上位词/下位词）。

### 1b. 核心文献定位

用精炼后的关键词运行 3-5 个检索式，定位核心文献：

```
"[精炼概念]" "[精炼结果]" "[学科]"
"[精炼概念]" "[机制词]" "[人群]"
"[精炼概念]" "[理论名称]"
"[主要概念]" 2022 OR 2023 OR 2024 OR 2025 OR 2026
"[主要概念]" debate OR critique OR challenge
```

详细检索策略见 [search-strategies.md](references/search-strategies.md)。

### 1c. 摘要总结

对前 5-10 篇高相关度论文，使用 WebFetch 获取摘要或全文信息。凡无法获得摘要的论文，只能列入待核验，不得进入种子论文。产出：

- **领域共识**：2-3 句话总结该领域已确立的发现
- **核心争论**：1-2 个主要争议点
- **关键词清单**：中英文对照的关键词变体表（用于后续 CNKI/Google Scholar 搜索）
- **种子论文**：5-10 篇核心论文的作者/年份/标题/关键发现
- **摘要清单**：每篇种子论文的摘要状态和2-3句摘要要点

**每次搜索后立即追加日志：**
```bash
cat >> "$SEARCH_LOG" << ROW
| [序号] | WebSearch | [精确检索式] | [返回数] | [有摘要保留数/待核验数] | [作者 年份; 摘要要点; 核心发现] |
ROW
```

### 1Q. 用户方向确认

完成 WebSearch 后必须 ask_user。重点让用户确认：

- 哪些关键词是有效方向；
- 哪些论文/子领域明显偏题；
- 下一步应优先查理论、机制、人群、方法还是中文文献缺口。

---

## Step 2：知识图谱（如有）

```bash
SHARED_REF_DIR="${PAPER_SHARED_REF_DIR:-${SCHOLAR_SKILL_DIR:-.}/shared-references}"
KG_REF="$SHARED_REF_DIR/knowledge-graph-search.md"
if [ -f "$KG_REF" ]; then
  eval "$(cat "$KG_REF" | sed -n '/^```bash/,/^```/p' | sed '1d;$d')" 2>/dev/null
  if kg_available; then
    echo "=== 知识图谱：主题搜索 ==="
    kg_search_papers "[主题]" 20 | kg_format_papers
    echo "=== 知识图谱：理论搜索 ==="
    kg_search_concepts "[主题]" 10 theory
  fi
fi
```

**目的**：补充 Step 1 可能遗漏的预提取发现和论文间关系。来源标签 `knowledge-graph`。

### 2Q. 用户方向确认

完成知识图谱阶段后必须 ask_user。重点让用户确认：

- 知识图谱补出的理论/概念关系是否符合研究意图；
- 哪些关系只是概念邻近但不应纳入；
- 下一阶段本地文献库应优先检索哪些作者、理论或机制词。

---

## Step 3：本地文献库（如有）

**目的**：用 Step 1 精炼后的关键词搜索本地已收集的文献，避免重复发现已知工作，并利用已存储 PDF 进行深度阅读。

**Zotero 选择规则**：如果用户在 Step 0Q 明确暂缓 Zotero/Zotero MCP，跳过本步骤并在日志中记录 `本地文献库=用户明确暂缓`。不得要求用户为了不保存全文的轻量检索安装 Zotero。若用户选择启用 Zotero/Zotero MCP 但工具不可用，记录为 `能力缺失`，提示安装配置，但继续后续在线检索。

**Zotero MCP 使用规则**：仅当用户选择启用 Zotero MCP，且当前宿主已按安装说明配置好可用的 Zotero MCP 实现时，才使用该实现检索和深读本地条目。本 skill 不假设固定工具名；若 Zotero MCP 不可用，再尝试 Zotero Connector、本地导出、BibTeX/EndNote 等方式。

**必须在单次 Bash 调用中完成**（shell 变量不跨调用持久化）：

```bash
SHARED_REF_DIR="${PAPER_SHARED_REF_DIR:-${SCHOLAR_SKILL_DIR:-.}/shared-references}"
eval "$(cat "$SHARED_REF_DIR/refmanager-backends.md" | sed -n '/^```bash/,/^```/p' | sed '1d;$d')" 2>/dev/null
echo "检测到文献源：$REF_SOURCES (主: $REF_PRIMARY)"

# 用 Step 1 精炼后的多组关键词搜索
echo "=== 关键词搜索：[精炼关键词1] ==="
scholar_search "[精炼关键词1]" 25 keyword | scholar_format_citations
echo "=== 关键词搜索：[精炼关键词2] ==="
scholar_search "[精炼关键词2]" 25 keyword | scholar_format_citations
echo "=== 关键词搜索：[精炼关键词3] ==="
scholar_search "[精炼关键词3]" 25 keyword | scholar_format_citations
```

**如果未检测到任何文献管理工具**（Zotero/Mendeley/BibTeX/EndNote），跳过此步，继续 Step 4。若用户原本选择启用 Zotero/Zotero MCP，状态记为 `能力缺失`；若用户选择轻量检索，状态记为 `用户明确暂缓`。

**通读 PDF**：对 Step 1 种子论文中匹配到的本地 PDF，提取摘要+引言：

```bash
if [ -n "$PDF_PATH" ] && [ -f "$PDF_PATH" ]; then
  pdftotext "$PDF_PATH" - | head -300
fi
```

**每次本地查询后追加日志：**
```bash
cat >> "$SEARCH_LOG" << ROW
| [序号] | 本地文献库 | [检索词] | [命中数] | [有摘要保留数/待核验数] | [作者 年份; 摘要要点; ...] |
ROW
```

### 3Q. 用户方向确认

完成本地文献库阶段后必须 ask_user。重点让用户确认：

- 已有文献是否足以覆盖核心理论；
- 是否需要优先深读某些本地PDF；
- 最终 CNKI/Scholar 应该补中文应用、国际经典、近年实证还是方法文献。

---

## Step 4：Annual Reviews 检查点

1. 搜索：`site:annualreviews.org "[Step 1 精炼主题]" review`
2. 优先：Annual Review of Sociology → Political Science → Public Health → Linguistics
3. 提取：经典基础文献、核心理论争论、近 5 年前沿
4. 至少添加 5 篇有摘要或可替代摘要的文献到工作书目

### 4Q. 用户方向确认

完成 Annual Reviews 阶段后必须 ask_user。重点让用户确认：

- 经典脉络是否应作为综述主线；
- 是否需要转向某个理论争论；
- 哪些前沿方向值得最后精准补充。

---

## Step 5：引文链扩展（模式 A/D）

- **向后**：检查前 5 篇高引论文的参考文献，识别多次出现的经典基础著作
- **向前**：对 2-3 篇基础论文，通过 Semantic Scholar API 查谁引用了它们
- **摘要要求**：引文链扩展得到的论文也必须补摘要；无法补摘要则只列为待核验。

```bash
PAPER_DOI="10.xxxx/xxxxxx"
curl -s "https://api.semanticscholar.org/graph/v1/paper/DOI:$PAPER_DOI?fields=citations.title,citations.year,citations.authors,citations.abstract&limit=20" | python3 -m json.tool
```

### 5Q. 用户方向确认

完成引文链扩展后必须 ask_user。该次确认决定最终 CNKI/Google Scholar 精准补充的检索式。必须向用户列出：

- 还缺中文应用、国际经典、近五年前沿、方法论文还是机制论文；
- 推荐的3-5个最终精准检索式；
- 预计每个检索式只补多少篇。

---

## Step 6：CNKI / Google Scholar 可达性检查（最终精准补充前）

此步骤只在用户已确认最终补洞方向后执行。CNKI 和 Google Scholar 不得默认二选一，也不得因模式 C/E、网络猜测或时间节省被静默跳过。

**CNKI 不可替代协议**：CNKI 的可达性、检索和完成状态只能来自 ZCode 内置浏览器控制中的 CNKI（kns8s）网页操纵结果。WebSearch、Google Scholar、普通搜索引擎、`cnki-researcher` 或 lit agents 只能作为关键词准备和策略建议来源，不得替代 CNKI 可达性检查、不得标记为 CNKI 完成状态、不得填充 CNKI 论文清单字段。

### Step 6.0：浏览器控制可用性检查（CNKI 硬闸门）

在访问 CNKI 检索页之前，先用 ZCode 内置浏览器控制做轻量检查；该检查通过前，不得执行任何 CNKI 检索、结果解析、详情页摘要抓取或下载。

1. **工具小测试**：列受控标签页或新建标签页，导航到 `about:blank` 或 `https://kns.cnki.net`，读取当前 URL/title 轻量状态。能列页/新建页/导航/读取，才算工具层可操控。
2. **登录预检**：打开检索页后检查页头机构信息（"大学/学院名 + 手机号"）。未登录时提示用户先在浏览器面板完成机构登录；未登录只能检索题录，不能下载全文。
3. **失败收敛**：工具抛错、无法列页/新建页/导航时，记录 `浏览器控制不可用`，立即停止 CNKI 阶段，提示用户重启宿主会话；不得用 WebSearch/Scholar/`cnki-researcher` 替代。
4. **通过后顺序**：先记录 `浏览器控制正常`，再按 [cnki-kns8s-closed-loop.md](references/cnki-kns8s-closed-loop.md) 打开 `https://kns.cnki.net/starter/advanced` 并执行一轮极轻量可达性测试。返回 `ok` 或明确 `zero_results` 表示页面操纵链路可用；真实可见验证码进入人工流程；`page_error` 记录 `CNKI 页面未完成`，不得误报验证码。

**快照噪音规则**：读取页面 DOM 快照或表单时可能出现隐藏的腾讯验证节点（如 `"拖动下方拼图完成验证"`、`"安全验证"` 文本）。判定验证码只允许使用 [cnki-kns8s-closed-loop.md](references/cnki-kns8s-closed-loop.md) 的几何可见性判据（验证码容器 `getBoundingClientRect().y > -1000 && height > 100`）；DOM 文本命中但几何判断为 false 的一律记为 `captcha_preloaded_hidden`，不得进入验证码流程、不得打扰用户。

### 检查方法

用浏览器控制先后访问两个站点，分别记录状态：

```
访问 https://scholar.google.com
  → 能加载（页面含 "Google Scholar" 文本）→ Google Scholar = 可执行
  → 无法加载（超时/DNS 错误/访问限制）→ Google Scholar = 网络不可达
  → 用户在 Step 0Q 明确暂缓 → Google Scholar = 用户明确暂缓

访问 https://kns.cnki.net
  → Step 6.0 小测试通过，能加载 CNKI 页面 → CNKI = 可执行
  → 验证码/登录阻断 → 按验证码/登录流程 ask_user，用户放弃才记为网络不可达
  → 浏览器控制可列页/新建页/导航且检索页可达 → 浏览器控制正常，继续 CNKI
  → 工具抛错/无法列页/新建页/导航 → 浏览器控制不可用，停止 CNKI
  → 页面已打开但检索/详情/摘要未完成 → CNKI = CNKI 页面未完成
  → 无法加载（超时/DNS 错误/访问限制）→ CNKI = 网络不可达
  → 用户在 Step 0Q 明确暂缓 → CNKI = 用户明确暂缓
```

**状态铁律**：Step 6 完成后，搜索日志必须同时出现 CNKI 和 Google Scholar 两行状态。CNKI 临时状态只能是 `可执行` / `用户明确暂缓` / `网络不可达` / `浏览器控制正常` / `浏览器控制不可用` / `浏览器页面未完成` / `CNKI 页面未完成`。若 Step 6.0 未通过，CNKI 不得进入 Step 7。若 CNKI 为 `浏览器控制正常`，继续网页操纵。若 CNKI 为 `浏览器控制不可用`，必须停止 CNKI 阶段并提示用户重启宿主会话；不得启动 WebSearch、Scholar 或代理替代。若任一来源为 `可执行`，Step 7 必须执行该来源的精准补充；若两者均不可执行或暂缓，必须 ask_user 请求下一步安排。

### 检测脚本（示意）

```javascript
async () => {
  const checks = [
    { source: 'Google Scholar', url: 'https://scholar.google.com', marker: 'Google Scholar' },
    { source: 'CNKI', url: 'https://kns.cnki.net', marker: '中国知网' }
  ];
  return checks.map(x => ({
    source: x.source,
    url: x.url,
    requiredStatus: '可执行 / 用户明确暂缓 / 网络不可达 / 浏览器控制正常 / 浏览器控制不可用 / 浏览器页面未完成 / CNKI 页面未完成',
    note: '用浏览器控制实际导航后填写，不得根据经验猜测'
  }));
}
```

### Step 7 后状态回写

精准补充完成后，必须回写 `SEARCH_LOG` 的"检索阶段预确认"表：

- 可达且完成 CNKI 网页检索、结果页解析和拟保留论文详情摘要抓取的来源：`CNKI 已执行`，并记录检索式、命中数、保留数、待核验数。
- 浏览器控制可列页/新建页/导航且检索页可达：`浏览器控制正常`，继续网页操纵。
- 工具抛错、无法列页/新建页/导航：`浏览器控制不可用`，停止 CNKI，提示用户重启宿主会话；不得替代检索。
- 页面已打开但检索、结果页、详情页或摘要抓取未完成：`CNKI 页面未完成`，不得写成已执行。
- 用户明确要求暂缓的来源：`用户明确暂缓`，并记录用户原话或简要原因。
- 完成可达性检查但无法访问，或验证码/登录阻断且用户选择放弃的来源：`网络不可达`。

不得把 `可执行` 留作最终状态；`可执行` 只是 Step 6 到 Step 7 之间的临时判断。
不得用 WebSearch、Google Scholar、普通搜索、`cnki-researcher` 或 lit agents 的结果把 CNKI 状态改写为完成状态。

---

## Step 7a：Google Scholar 最终精准补充（可达且未暂缓时）

导航到：`https://scholar.google.com/scholar?q=URL编码查询词`（空格 → `+`）

**只使用用户在最后一次阶段确认中认可的精准补洞检索式。** 每个检索式默认保留 3-5 篇，除非用户要求扩展。Google Scholar 结果页片段不等于摘要，只能用于候选排序；所有拟保留论文必须继续打开来源页、期刊页、数据库页或 PDF 抓取摘要/等价全文摘要信息，无法获得摘要则列入待核验。

```javascript
async () => {
  await new Promise((r, j) => {
    let n = 0;
    const c = () => {
      if (document.querySelector('#gs_res_ccl_mid') || document.querySelector('.gs_r')) r();
      else if (++n > 20) j('timeout');
      else setTimeout(c, 500);
    };
    c();
  });

  const results = Array.from(document.querySelectorAll('.gs_r.gs_or.gs_scl')).map((row, i) => {
    const titleEl = row.querySelector('.gs_rt a, h3.gs_rt a');
    const snippetEl = row.querySelector('.gs_rs');
    const metaEl = row.querySelector('.gs_a');
    const citedEl = row.querySelector('.gs_fl a:nth-child(3)');
    const metaText = metaEl?.innerText?.trim() || '';
    const authorMatch = metaText.match(/^(.+?)\s*[-–]/);
    const yearMatch = metaText.match(/(\d{4})/);

    return {
      n: i + 1,
      title: titleEl?.innerText?.trim() || '',
      href: titleEl?.href || '',
      snippet: snippetEl?.innerText?.trim() || '',
      authors: authorMatch ? authorMatch[1].trim() : '',
      year: yearMatch ? yearMatch[1] : '',
      citedBy: citedEl?.innerText?.match(/(\d+)/)?.[1] || ''
    };
  });

  const totalText = document.body.innerText.match(/About ([\d,]+) results/);
  return { source: 'Google Scholar', total: totalText ? totalText[1] : '0', results };
}
```

翻页/排序按 [cnki-kns8s-closed-loop.md](references/cnki-kns8s-closed-loop.md) 的排序控件（`li#FFD` 相关度 / `li#CF` 被引 / `li#DFR` 下载）执行。

---

## Step 7b：CNKI 最终精准补充（可达且未暂缓时）

默认导航到新版检索页，且正式 CNKI 检索只默认这一入口（进入后切换"专业检索"标签）：

```text
https://kns.cnki.net/starter/advanced  →  https://kns.cnki.net/kns8s/AdvSearch
```

**可用性闸门**：执行本步骤前必须已经完成 Step 6.0。若未记录 `浏览器控制正常`，不得打开检索页；若记录为 `浏览器控制不可用` 或 `CNKI 页面未完成`，立即停止 CNKI 阶段并提示用户重启宿主会话或处理页面阻断。不得用 WebSearch、Scholar、`cnki-researcher` 或 lit agents 继续冒充 CNKI。

**只使用用户在最后一次阶段确认中认可的精准补洞检索式。** 每个检索式通常保留 3-5 篇，第一轮只用单一主概念的宽松专业检索式（`SU='主概念'`），同义概念用 `+` 并入，不叠加 `*` 跨概念、来源类别、年份、作者或期刊。结果过大时先点「学术期刊N」筛选，再按被引排序（`li#CF`）；筛选后若 0 条，立即回退宽检索。每轮 CNKI 检索返回后，必须在阶段确认中显式报告命中总数、是否触发筛选收窄。CNKI 搜索结果页的标题、作者、期刊、下载量和被引量只用于候选排序；拟保留论文必须进入详情页抓摘要。

**CNKI 入口铁律**：正式 CNKI 检索一律使用专业检索页 `https://kns.cnki.net/starter/advanced`（自动跳转 `kns.cnki.net/kns8s/AdvSearch` 后点击「专业检索」标签）。专业检索是默认入口，不需要使用基础检索框、AI 检索或其他 CNKI 页面。基础检索框只能用于单个自然短语、专名或站点可达性临时测试；不得把 WebSearch/Google Scholar 风格的布尔串粘进基础检索框。若用户给出多个中文关键词，先转换为专业检索式（同义概念并入同一 `SU=(...+...)` 组，概念间用 `*`），而不是直接填入基础搜索框。

**CNKI 来源铁律**：Step 7b 的所有结果必须来自 CNKI 网页操纵。`cnki-researcher`、search-strategy agent、WebSearch 和 Google Scholar 只能提供检索式建议，不得生成 CNKI 结果、不得声明 CNKI 完成、不得替代详情页摘要抓取。若 CNKI 页面、浏览器控制、验证码、登录或机构访问阻断未解决，记录对应未完成状态并停止 CNKI 阶段。

### 7b-0. 选择 CNKI 子流程

完整协议见 [cnki-kns8s-closed-loop.md](references/cnki-kns8s-closed-loop.md)。执行时按当前子流程只读取协议对应章节，避免一次性加载全部代码。

| 需求 | kns8s 流程 | 说明 |
|------|----------|------|
| 结果量校准 | 专业检索 + 筛期刊 | 用 `.pagerTitleCell em` 读取命中总数判断结果量；点击「学术期刊N」筛选只保留期刊论文 |
| 分阶段筛选 | 期刊筛选 + 排序 | 结果过大时先筛期刊，再按被引排序（`li#CF`）；跨概念 `*`、年份、作者或期刊限制需用户确认 |
| 扩展样本 | 双排序 | 政策阐释类按被引（`li#CF`）取经典，实证类按相关度（`li#FFD`）取新作 |
| 深读核心文献 | 页内 fetch 详情 | 只对高相关/高被引/最新关键论文抓摘要、关键词、基金、PDF 链接 |
| 下载全文 | Cookie + curl | `scripts/cnki/kns8s-download.sh` 免弹窗批量下载，禁止浏览器下载（必弹 Save-As） |
| 期刊核验 | 期刊检索/收录查询 | 检查期刊来源、CSSCI/北大核心等收录状态 |

文献综述默认策略：每组关键词必须先用专业检索做宽松检索确认结果量，再只抓取与已确认缺口直接相关的结果；对每轮保留论文全部做详情提取以获得摘要；如需要进入 Zotero，用知网导出题录而不是逐篇复制。题录导出只保存元数据，不替代摘要抓取。

**多关键词处理规则**：多个中文关键词必须先拆成“概念组”。同义词、近义词、上下位词、译名差异属于同一概念组，用专业检索 `+`（或）并入同一 `SU=(...)`；不同概念组先分轮宽检索，不要自动并列为必含条件。不得自动用 `*`（与）并列全部概念；只有当宽检索结果量过大、且用户确认需要跨概念收窄时，才追加 `*` 概念、年份、作者或期刊限制，并把收窄理由写入日志。

**概念组示例**：

| 用户输入 | CNKI 专业检索处理 |
|---|---|
| `社会资本 劳动力市场 流动 分层` | 第一轮分别用 `SU='社会资本'`、`SU='劳动力市场'` 宽检索；`流动/分层` 作为机制或结果方向，结果过大且用户确认后再追加 |
| `不平等 差距 分层` | 视为同一概念组，`SU=('不平等' + '差距' + '分层')` |
| `共同富裕 收入分配 城乡差距` | 先检索 `SU='共同富裕'`；需要扩展时用 `SU=('共同富裕' + '收入分配' + '城乡差距')` 作为相关概念组，不默认设为必含 |

**无结果处理规则**：如果追加筛选后返回 `zero_results` 或 `total = 0`，记录为 `检索式过窄/无命中`，并回退到上一轮宽检索条件；不得继续追加限制。不得把无结果、结果表为空、页面未加载完或未命中提示误判为验证码。只有通过 [cnki-kns8s-closed-loop.md](references/cnki-kns8s-closed-loop.md) 的几何可见性判据确认验证码容器真实可见时，才进入验证码流程；`captcha_preloaded_hidden` 只记录诊断，不询问用户。

**分阶段筛选**：完整协议见 [cnki-kns8s-closed-loop.md](references/cnki-kns8s-closed-loop.md)。每次检索动作记录 `status: "ok" | "zero_results" | "captcha" | "page_error"`、实际启用的筛选字段，以及检索式、命中数、验证码状态、`fallbackAction`，便于区分检索式过窄、隐藏验证码预加载和页面阻断。

**读取摘要**：对所有拟保留论文，在检索结果标签页内用 `fetch(href, { credentials: 'include' })` + `DOMParser` 页内解析（每批 ≤3 个 URL），提取 `#ChDivSummary` 摘要、`p.keywords a` 关键词、`a#pdfDown` PDF 链接，完整片段见 [cnki-kns8s-closed-loop.md](references/cnki-kns8s-closed-loop.md)。无摘要返回 `abstractStatus: "无摘要-不得纳入"`，只能列入待核验，不得纳入证据。

### 验证码处理（强制用户手动验证流程）

**核心原则：检测到验证码时，必须请用户在浏览器面板中手动完成，不得跳过或用 WebSearch 替代。**

**处理流程：**

```
检测到验证码（几何可见性判据为 true：验证码容器在视口内且尺寸有效）
  → 1. 停止自动化操作（浏览器面板本身对用户可见，无需前置窗口）
  → 2. 用 ask_user 询问用户"验证码是否已完成？"
       - 选项 A："已完成，继续搜索" → 重试当前搜索步骤
       - 选项 B："未完成，还需要时间" → 等待 15 秒后再次截图询问
       - 选项 C："放弃 CNKI 搜索" → 终止 CNKI 操作
  → 4. 用户选择 A 后，立即重新执行当前被阻断的搜索操作
```

验证码阻断的结构化示例：

```text
question: "CNKI 页面出现可见验证码或登录阻断。请在浏览器面板完成拖动验证后选择下一步。"
header: "CNKI验证"
options: [
  {label: "已完成，继续", description: "重试当前 CNKI 搜索、翻页、详情页或摘要抓取步骤"},
  {label: "还需时间", description: "等待 15 秒后重新截图，并再次询问用户"},
  {label: "放弃CNKI", description: "终止 CNKI 操作，记录为用户放弃或网络不可达"}
]
```

**检测方法**：每次页面加载、搜索、翻页、详情或下载后，统一使用 [cnki-kns8s-closed-loop.md](references/cnki-kns8s-closed-loop.md) 的几何可见性判据（`#tCaptchaDyMainWrap` 容器 `getBoundingClientRect().y > -1000 && height > 100`）。`captcha_preloaded_hidden` 表示预加载未激活，继续检索或按 `zero_results` / `page_error` 处理；只有几何判据为 true 才进入人工流程。

**快照噪音处理**：不得仅凭 DOM 快照、`document.body.innerText`、隐藏 DOM 文本或节点文本 "拖动下方拼图完成验证" 判定验证码。真实验证码必须满足 [cnki-kns8s-closed-loop.md](references/cnki-kns8s-closed-loop.md) 的几何可见性判据（容器在视口内且尺寸有效）。

**禁止行为：**
- ❌ 检测到验证码后直接改用 WebSearch 替代
- ❌ 自动刷新页面尝试绕过
- ❌ 不告知用户就跳过 CNKI 搜索
- ❌ 把 DOM 快照中不可见的 `您当前IP` / `安全验证` / `拖动下方拼图完成验证` 当作真实页面内容

---

## 搜索后评估

对照模式目标：
- **模式 A**：40-80 篇 | **模式 B**：15-30 篇 | **模式 C**：10-20 篇
- **模式 D**：30-60 篇 | **模式 E**：10-30 篇

不足时不得自动继续堆积。必须先 ask_user，说明缺口、缺少的摘要、建议追加来源和检索式，经用户确认后再追加。

搜索不足时的结构化示例：

```text
question: "当前论文数量或摘要覆盖不足。是否按建议追加来源和检索式？"
header: "补检确认"
options: [
  {label: "按建议补检", description: "使用当前缺口报告中的 1-3 个追加来源或检索式继续搜索"},
  {label: "用户指定补检", description: "用户指定新关键词、来源、理论、机制或人群方向"},
  {label: "停止搜索", description: "保存当前日志和论文清单，记录为用户确认停止"}
]
```

**完成所有搜索后追加最终论文清单：**
```bash
cat >> "$SEARCH_LOG" << 'SNAP'

### 最终论文清单（全部搜索完成后）— [N]篇总计

| 作者 | 年份 | 标题 | 期刊 | 摘要状态 | 摘要要点 | 方法 | 来源 | 相关度 |
|------|------|------|------|----------|----------|------|------|--------|
SNAP
```

---

## Step 8：CNKI kns8s 精准检索闭环（实测修正协议，2026-09-05）

> 来源：2026-09-05 实跑复盘。旧协议的失败模式与修正一一对应，执行时不得回退到旧做法。

### 8.1 操作修正（点击与页面定位）

| 失败模式（旧） | 修正（必须执行） |
|---|---|
| `getByRole("listitem")` 点击「专业检索」无响应 | 切换标签用 `evaluate(() => document.querySelector('li[name="majorSearch"]').click())`——kns8s 的检索方式标签是 `li[name=gradeSearch|majorSearch|authorSearch|sentenceSearch]`，点击后 `li.className` 变为 `active` 即成功 |
| 结果过大时误点隐藏面板按钮、或连续多次点错检索按钮 | 专业检索面板提交按钮是 **`#ModuleSearch input.btn-search`（可见）**；`input.search-btn` 是隐藏镜像按钮，Playwright 真实 click 会超时。先 `fill()` 到 `textarea.majorSearch`，再唯一化定位 `#ModuleSearch input.btn-search` 并 `isVisible()` 确认后点击 |
| 反复轮询"检索-中国知网"结果标签页但内容为空 | **结果直接渲染在 AdvSearch 主页面 body 内**（"共找到 N 条结果" + 结果表），不是新标签页。读结果的正确姿势：`document.body.innerText` 中定位 `共找到` 与 `题名` 表头；结果页 iframe 里的"安全验证/拖动拼图"文案按几何可见性判据检查（可见容器数=0 即隐藏预加载，不构成闸门） |
| 结果表解析用 `table.querySelectorAll('tr')` 取到 undefined | 表格可能延迟渲染：先 `waitForTimeout(3500)`，再从 `body.innerText` 的 `题名`/`篇名` 表头切片读取；`innerText` 方式对表头为"题名"（总库）或"篇名"（期刊）都要兼容 |
| 学术期刊筛选点 span 无效 | 筛选入口是 facet 列表中含计数的祖先可点元素：找文本以"学术期刊"开头的可见 span 后，向上遍历到 `A/LI` 祖先再 click；点击后计数文本变为"学术期刊N"同款格式（如"学术期刊165"） |
| "被引"排序入口文本定位失败 | 排序条目在"排序："行内（相关度 发表时间 被引 下载 综合）；定位失败时改从 `body.innerText` 确认当前排序名，重试用坐标兜底（`cua.click`）或刷新后重试一次，仍失败则记录"被引排序不可用"并只保留相关度序 |

### 8.2 结果量分级纪律（>1000 触发讨论）

1. 任一轮命中 **>1000** 时，必须先做**学科边界不清讨论**：逐个检查命中学科分布（结果页"学科"facet），判断检索词是否混入无关学科（如"分类控制"命中煤炭/医学、"嵌入"命中教育技术）。若确认边界不清：先收窄字段（SU→TI）或叠加组织概念词，再评估。
2. 字段收窄后仍 >1000 且学科边界清晰（如"政府购买×社会组织"本身就是跨管理/社会学/法学的正当交叉领域）：**不加学科限制，改用来源类别筛选**——依次点选 **CSSCI、北大核心、AMI 核心** facet（可多选；结果页显示为"北大核心(N) CSSCI(N) AMI(N)"），把池子压到 ≤300 再进入摘要抓取。
3. 命中 100–1000：默认点"学术期刊"facet 即可。命中 <100：直接进入摘要抓取。
4. 每次筛选动作后必须显式报告新命中数（写入搜索日志的筛选动作列）。

### 8.3 双排序与首页摘要抓取

- **相关度排序与被引排序都必须执行**（被引排序入口见 8.1 修正；不可用时记录）。两种排序各取第一页（20 条）。
- 第一页的**每一条**都必须打开详情页读取并记录摘要：从结果表题名链接取 `href`（`kcms2/article/abstract?...` 形态），逐条 `goto` → 等 domcontentloaded → 从详情页提取 题名/作者/机构/期刊/年/被引/下载/摘要/关键词/DOI。摘要铁律在此环节闭环：抓到的摘要立即写入搜索日志论文清单并标【摘要已核】。
- 详情页可顺带记录 **PDF 下载直链**（页内"PDF下载"按钮的 href 或 onclick 中的 URL），登记到 paper-registry.csv 对应条目的 download_url 字段——这是 Step 9 的输入。
- 批量节奏：每 5–8 篇摘要落盘一次（追加写搜索日志），防上下文压缩丢失。

### 8.4 引证文献（前沿）与共同参考文献（学科基础）

执行顺序（试错后定序）：先抓首页摘要建立池子 → 从池中挑 **被引最高的 2–4 篇锚文献** → 各自打开详情页：
1. **引证文献 tab**：按被引/发表时间倒序取最新的 5–10 条——作为**研究前沿**（谁在最近引用它、引用方向是什么）；新题录进入池子候选。
2. **参考文献/相似文献**：跨锚文献重复出现的条目——作为**学科基础**（共同引用文献）；缺失的基础文献必须补入池子并抓摘要。
3. 锚文献选择优先中英文各至少 1 篇，防止前沿与基础都偏单语。

### 8.5 CNKI 状态回写

每轮结束在搜索日志记录：检索式、命中数、筛选动作（学术期刊/CSSCI/北大核心/AMI）、双排序状态、首页摘要已抓数、引证/共引执行状态、失败与兜底动作。

## Step 9：Top-N 归档与下载状态（模式 A/B/D，N 按论文规模定，期刊论文建议 30–50）

### 9.1 选择

- 从“摘要已核”池中按相关度、被引、前沿和基础标记综合判选 Top N；将判选理由写入 paper-registry.csv 的 selection_reason。下载前每篇都必须先通过 literature_registry.py register 获得稳定 paper_id、项目内 pdf_path 和 fulltext_path。

### 9.2 下载（CNKI）

- 用 8.3 存下的 PDF 直链登记到注册表，再生成下载计划。计划的列为 paper_id、pdf_path、download_url，文件名和最终目录不得手工指定。
- 浏览器详情页导出 Cookie 后，使用 scripts/cnki/kns8s-download.sh。脚本下载至 .part，完成本地 PDF 解析校验后才原子归档，并把哈希、页数和状态写回注册表。
- 单批最多 15 篇、每篇间隔 6 秒。首次出现 Cookie/令牌失效、验证码或频控页时立即停止；剩余条目保留在注册表中，等待用户在可见页面完成操作或冷却后生成新计划续跑。

### 9.3 下载（外文，试错路径按优先级）

1. **开放获取直查**：Unpaywall API（`https://api.unpaywall.org/v2/<DOI>?email=...` 的 `best_oa_location.url_for_pdf`）、Semantic Scholar openAccessPdf、出版社 OA 页。
2. **Sci-Hub 系镜像**：依次试 `sci-hub.se` / `.st` / `.ru`（`<镜像>/<DOI>` 跳转后取 `<iframe src>` 或 `embed` 处 PDF URL，curl 直下；镜像失效/验证码即换下一个）；下载失败不重试超过 2 个镜像。
3. 全部失败 → 在注册表标记 retry、failed 或 blocked，并附失败原因；绝不伪造下载成功。

### 9.4 注册表与下载计划

唯一正式记录是 paper-workspace/02-literature/paper-registry.csv。它保存题录、paper_id、下载 URL、项目内 PDF 路径、哈希、页数、解析路径和各阶段状态。下载计划是从注册表即时生成的临时 TSV，不能作为第二份事实来源：

```bash
OUTPUT_ROOT="${OUTPUT_ROOT:-paper-workspace}"
PLAN="${OUTPUT_ROOT}/02-literature/plans/download-plan.tsv"
python3 scripts/literature_registry.py download-plan \
  --workspace "${OUTPUT_ROOT}" --out "${PLAN}"
bash scripts/cnki/kns8s-download.sh \
  /tmp/cnki_cookie.txt "${OUTPUT_ROOT}" "${PLAN}"
```

旧项目中的 downloads/、cnki-fulltext/ 和 download-manifest.* 只作为迁移输入；登记 source_pdf_path 或已有 pdf_path 后保留原文件，不自动搬移。

### 9.5 人机协作与风控实测教训（2026-09-06 第二轮实跑沉淀，强制执行）

1. **order 令牌一次性**：详情页"PDF下载"链接内的 order 令牌消费一次即失效；被消费后再点击/请求一律报"来源应用不正确(01)"。**自动化诊断（curl 试令牌）与人工点击严禁共用同一页面令牌**——主流程提取令牌做诊断前，必须确认用户当前不依赖该页面做人工下载；人工点击模式下单流程只导航到详情页、绝不提取令牌。
2. **频控阈值**：cookie+curl 连续下载约 24 次（间隔 2s）后触发拼图校验（302 → bar/verify/index.html，errorcode=3）。规程：单会话批量下载 ≤15 篇、每篇间隔 5–8s、两批之间 ≥30 分钟；出现第一个拼图页立即停止自动化，不得重试。
3. **拼图页不可脚本化通过**：拼图校验页（"验证成功，请稍等"）只是中间跳转页，真正的拼图必须由真人在可见浏览器面板完成；curl 跟随 returnUrl+vLevel 只会得到 (01) 或 (0315 会话结束)。正确人工流程：真人点击详情页"PDF下载"→完成拼图→风控等级下降→**重新导出 cookie→先测 1 个新令牌→成功才恢复批量**。
4. **风控期内全部全文路由被封**：PDF 下载、HTML阅读、原版阅读均经同一风控网关，标记状态会持续数十分钟至小时级；此时不得反复重试，把剩余条目在注册表标记为 blocked 并记录原因，给出续跑协议，先推进已归档论文的核读与证据整理。
5. **外文获取实录**：Unpaywall 命中率对 CQ/VOLUNTAS 类中国研究约 20–30%（本轮 11 DOI 命中 1）；sci-hub 镜像（.se/.st）对 2020+ 中国研究论文基本未缓存（2 镜像试错后即标 `未获取-镜像未缓存`）。已知机构库 OA（LSE Research Online、UMN conservancy、CUHK）是最高产通道——exa 检索时同步记录机构库链接。
6. **MinerU 实录**：混合来源 PDF 可逐篇补跑；转换只接受注册表中 download_status=downloaded 的 PDF，.part 与其他失败残留不得进入解析器。
7. **被引排序易失效**：`li#CF` 点击需在结果完全加载后 ≥4.5s 才生效，且新检索会话中可能静默回落相关度序——**每次排序后必须用首条题名验证**（对照已知高被引文献），未生效则重试一次，仍失败则本轮只保留相关度序并记录。
8. **风控衰减与续跑实测（validated）**：触发拼图后约 1–2 小时自然衰减；解除判定法=新会话取新令牌 curl 探测（返回 %PDF 即解除）。续跑实测 9/9 成功（间隔 6s）。定位续跑目标时，**被引排序 + 已知题名过滤**比相关度序可靠（相关度序对同一查询会漂移）；结果列表匹配不到目标时扩大 slice 到 20 条或翻页。
9. **MinerU 多文件传参坑**：一次调用传多个文件路径可能只处理部分；使用 --registry 时解析器会按 paper_id 隔离逐篇处理并回写状态。不得用下载编号或人工 ls 重新对应文件。

## Step 10：MinerU 全文化 → 全文综述（模式 A/B/D 终段）

唯一执行方式如下：

```bash
OUTPUT_ROOT="${OUTPUT_ROOT:-paper-workspace}"
python3 scripts/mineru/pdf2md.py "${OUTPUT_ROOT}/02-literature/papers" \
  --output "${OUTPUT_ROOT}/02-literature/fulltext" \
  --registry "${OUTPUT_ROOT}/02-literature/paper-registry.csv"
# 长文追加 --split-long；Token 失效且文件满足上限时可用 --agent
```

每篇解析结果固定为 fulltext/paper_id/document.md；图片保留在同一 paper_id 目录。解析器逐篇回写 parse_status 与 fulltext_path，部分失败不能写作全部完成。

全文核读应提取将进入正文的判断、原文位置、方法或文本依据、范围条件、可转述内容和不可声称内容，写入 review-evidence.csv。取消“过半主张来自全文”的比例要求：关键机制、方法批评、效应数值、争议判断和强空白判断必须逐条映射到全文或理论文本。
