# Paper 文献综述 4SS（lit 模块独立版）

中英文双语文献综述与假设推导一体化技能。支持五种模式：完整文献地图（A）、定向综述（B）、快速概览（C）、文献综述+假设推导（D）、知网专项搜索（E）。自动搜索本地文献库、CNKI 中文文献（ZCode 内置浏览器控制 kns8s 专业检索）、Google Scholar、WebSearch、Annual Reviews，生成结构化文献景观地图；收到理论、规范或阐释设计报告时在既有流程中组织支持立场、竞争立场和反例材料，不强制假设推导。当用户需要写文献综述、做系统回顾、找研究空白、提出研究假设、搜索中英文文献时使用。

本包由 `paper-master-4ss/scripts/export_standalone.py` 从总控包 `paper-master-4ss/modules/lit/` 自动导出：

- 包内相对路径相对本包根目录解析；
- `master/` 与 `references/` 中的协议/治理文件是导出时拷贝的快照；
- 跨模块路径 `paper-master-4ss/modules/<x>/...` 相对同级安装的总控包解析；
- 更新方式：修改总控包对应模块后运行
  `python3 paper-master-4ss/scripts/export_standalone.py lit` 重新导出，勿直接编辑本包。
