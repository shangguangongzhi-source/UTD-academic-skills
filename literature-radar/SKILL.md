---
name: "literature-radar"
description: "顶刊文献精准检索与分级筛选技能。当用户需要检索学术文献、查找研究主题相关论文、搜索顶刊文献、做文献综述的文献筛选、查找经济学/管理学/金融学核心期刊论文时触发。触发词包括但不限于：检索文献、搜文献、找论文、查文献、文献雷达、顶刊检索、核心文献、分级筛选、文献筛选、帮我找文献、搜一下相关研究。"
version: "2.0.0"
---

# 文献雷达 · 分级筛选技能

经管领域顶刊文献精准检索技能，从源头严格把控文献质量。

本技能分为两层：
- **静态层**（`static/`）：期刊等级体系、搜索规则等版本化内容，每次加载。
- **动态层**（本文件 + `manifest.yaml`）：检测用户需求，加载匹配的工作流，按需引用深度参考。

**不要从记忆或本文件直接应用检索逻辑。** 始终从磁盘加载所需片段。

---

## 路由协议

### 1. 加载 manifest 和核心层

读取 `manifest.yaml`。同时读取 `always_load` 中的所有文件：

- `static/core/journal-tiers.md` — 三级期刊等级体系（UTD24/ABS4*/NSFC A类/中文顶刊）
- `static/core/search-rules.md` — 真实性红线、验证流程、输出字段、数量控制

### 2. 检测工作流

将用户需求映射到一个或多个工作流值：

- `topic-search` — 按研究主题检索顶刊文献
- `author-tracking` — 按作者/团队追踪发表记录
- `citation-trace` — 引文网络追踪（前向/后向）
- `journal-scan` — 扫描特定期刊最新发表

组合请求可同时加载多个工作流。用一行文字说明检测到的工作流。

### 3. 加载匹配的工作流

读取 `manifest.yaml` 中工作流对应的文件。**不要**加载全部工作流。

### 4. 执行工作流

按以下顺序应用加载的内容：
1. 核心规则（期刊等级、搜索规则）
2. 工作流特定步骤
3. 按需加载的共享模块（去重引擎、检索策略）

**输出方式**：首选使用 `scripts/render_report.py` 渲染 HTML 报告（将结构化 JSON 数据传入脚本）。回退方案参照 `static/templates/html-report.md`。

### 5. 深度参考按需调用

`references/` 和 `scripts/` 下的文件按需打开：
- `references/dedup-engine.md` — 去重逻辑（多轮检索时使用）
- `references/search-strategy.md` — 关键词扩展与多轮检索策略
- `scripts/render_report.py` — HTML 报告渲染脚本