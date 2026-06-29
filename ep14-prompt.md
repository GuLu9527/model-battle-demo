# 模型挑战第 14 期 · 统一任务指令

> 给 9 个模型的**同一份 prompt**。最终结果由观众投票决定胜负，本期不评分。

---

你正在参加「模型挑战」第 14 期——一个 AI 模型公开竞赛节目。本期命题：**XCrawl 数据采集对决**。

## 你的任务

使用 **XCrawl API**（一个 URL→Markdown 的数据采集工具），完成两件事：

1. **抓取数据**：从 HuggingFace（https://huggingface.co/）上抓取顶尖开源模型的信息
2. **生成展示页**：把抓取的数据整理成一个可在浏览器打开的 HTML 页面

## 抓取目标

至少 **20 个**主流开源模型，每个模型包含以下字段：

| # | 字段 | 说明 |
|---|------|------|
| 1 | 模型名称 | 完整名称（如 `meta-llama/Llama-3-70B`）|
| 2 | 首次发布日期 | 最早版本的时间 |
| 3 | 最新版本发布日期 | 最新版本的时间 |
| 4 | 模型参数量 | 单位 B（billion）或 M（million）|
| 5 | 主要基准测试分数 | 至少 1 个（MMLU、HumanEval、HellaSwag 等）|

**字段缺失可以写 `N/A`，不要编造数据。**

## HTML 页面要求

- **模型列表表格**（可排序：点击表头排序）
- **至少 3 种排名维度**，例如：
  - 版本迭代最多（历史最悠久）
  - 参数规模最大
  - 综合能力最强
  - 纯文本能力最强
  - 多模态能力最强
- **含 CSS 样式**，可在浏览器直接打开
- **响应式**（手机/电脑都能看）

你可以自由选择：

- 配色 / 字体 / 布局
- 排名维度的具体定义和算法
- 字段的呈现方式（表格 / 卡片 / 图表）
- 抓取的模型数量（≥20 即可，多多益善）

## 本期特殊说明（重要）

**本期没有标准答案。**

AI 抓多少字段、HTML 怎么设计、排名维度选哪些——都是你的**主观选择**。
我们不评分。胜负由**观众投票**决定，评论区点赞最高的就是本期获胜者。

所以：

- ✅ 抓得**全**：覆盖更多模型和字段
- ✅ 做得**好看**：HTML 视觉冲击力强
- ✅ 排名**有逻辑**：维度定义清晰、计算合理
- ❌ 不要**瞎编**：宁可写"未知"也不要编造数据（观众一眼看穿）

## 输出

完整的 HTML 文件（含 CSS 样式），路径自定，建议保存为 `result.html`。

## XCrawl CLI 使用（v0.2.7 · 必须用 v0.2.7）

> 认证已完成（API Key 已存到 `~/.xcrawl/config.json`），直接跑命令即可。

### 1. 3 步抓取策略（按顺序执行）

**Step 1: 搜索排行榜页面**
```bash
xcrawl search "top open source LLM leaderboard huggingface" --limit 10 --format markdown
```
- 从返回结果挑 1-2 个 HuggingFace 排行榜页 URL
- 重点看 `huggingface.co/spaces/*leaderboard*` 这类空间

**Step 2: 抓取排行榜页（拿到模型名单）**
```bash
xcrawl scrape "https://huggingface.co/spaces/open-llm-leaderboard/open_llm_leaderboard" --format markdown --output leaderboard.md
```
- `--output <path>` 保存到文件，多 URL 时作为目录

**Step 3: 抓取 top 10 模型各自的 model card（拿完整 5 字段）**
```bash
xcrawl scrape \
  "https://huggingface.co/meta-llama/Llama-3-70B" \
  "https://huggingface.co/mistralai/Mistral-7B-v0.1" \
  "https://huggingface.co/Qwen/Qwen2.5-72B" \
  "https://huggingface.co/google/gemma-2-27b" \
  "https://huggingface.co/01-ai/Yi-1.5-34B" \
  --format markdown --output model-cards.md
```
- 至少抓 5 个 model card（多多益善）
- 每个 card 含参数、发布日期、基准分数

### 2. 让 AI 解析

拿到 markdown 文件后，让 AI 解析并生成 HTML 展示页。

### 3. 关键注意事项

- **不要直接 scrape `https://huggingface.co/`**（只拿到首页营销内容，没数据）
- **字段缺失写 `N/A`**，**不要编造**
- **批量并发**：`--concurrency 3`（多个 scrape 时建议）
- **输出格式**：`--format markdown`（默认）/ `html`（含结构）/ `json`（程序化）

---

> **录制要求和验收标准**在 `checklist.md`（仅给录制者看，不要贴给 AI）。
