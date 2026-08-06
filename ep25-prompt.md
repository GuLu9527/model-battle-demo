# EP25 Prompt — Qwen 3.8 Max vs DeepSeek V4 Flash vs GPT-5.6 Luna

## Q1 · OpenCode Go 使用数据 BI 大屏

## 背景

OpenCode Go 是 OpenCode（开源 AI 编程工具）的低成本订阅服务：首月 $5，之后每月 $10，提供对一批精选开源编程模型的稳定访问。本任务数据来自官方公开统计页（数据截至 **2026-08-04**），展示该服务的真实使用情况。

## 数据

数据目录 `data/` 下有 9 个 CSV 文件：

1. `usage_daily.csv` — 90 天各模型每日 token 份额（值域 0–1，如 0.8281 = 82.81%）
2. `users_daily.csv` — 90 天各模型每日活跃用户数（单位：人）
3. `market_author_share.csv` — 厂商（DeepSeek / Xiaomi / Zhipu 等）每日份额（%）与 token 总量（单位：T）
4. `country_distribution.csv` — 国家/大洲 token 分布（T 与 %）
5. `leaderboard.csv` — 1D / 1W / 1M / 3M 模型排行（tokens、rank、change）
6. `token_cost.csv` — 各模型 token 成本
7. `cache_ratio.csv` — 缓存命中率
8. `session_cost.csv` — 会话成本
9. `opencode-go-pricing.csv` — 订阅套餐定价（输入/输出/缓存价格，$/1M tokens；部分模型有两档价格，注意区分）

## 任务

做一个**单文件 HTML** 的交互式 BI 大屏（可视化库可用 CDN，如 ECharts / Chart.js）。必须包含：

1. 模型使用份额趋势（90 天时间序列，支持范围切换）
2. 活跃用户排行 Top 10
3. 厂商市场份额对比
4. 国家/地区分布
5. 成本 / 缓存 / 会话维度分析
6. 结合价格表的「性价比榜」（每个模型每 $1 能跑多少 token）
7. 页面显著位置标注：「数据截至 2026-08-04 · 来源 opencode.ai/zh/data」

## 注意

- 各表**单位不统一**（份额是 0–1 小数、token 量有 T 有 K、用户是人数），必须正确理解并在图表中正确标注单位
- 数据是真实数据，**不要编造或修改任何数值**
- 图表类型要适配数据：份额趋势用折线/面积图、排行用条形图、国家用地图或条形图
- 交互：至少支持时间范围切换与图表联动

## 输出

单个 HTML 文件，浏览器直接打开即可运行。

---

## Q2 · 百万行数据性能挑战

## 背景

`data/usage_log_1m.csv` 是 100 万行「OpenCode Go 订阅用户一个月使用日志」（合成数据，字段与真实场景一致）。

## 数据

`usage_log_1m.csv`（100 万行，11 列）：

- `id`：自增 ID
- `ts`：时间戳（2026-07 整月）
- `user_id`：用户 ID（1–5000）
- `model`：模型（18 种，按真实使用份额分布）
- `action`：操作类型（chat / edit / complete / refactor / debug / test / docs）
- `prompt_tokens` / `cache_tokens` / `completion_tokens`：本次请求 token 数
- `cost_usd`：本次请求成本（$）
- `region`：地区（按真实分布）
- `status`：ok / error / timeout

## 任务

做一个**单文件 HTML** 的大数据表格/视图，能流畅处理这 100 万行：

1. 渲染 + 滚动 100 万行不卡顿（**必须**用虚拟滚动 / Canvas / Web Worker 等优化手段，禁止把 100 万行 DOM 全量塞进页面）
2. 支持按 `model` / `region` / `status` / `action` 筛选
3. 支持关键字搜索（如 `user_id`、`region`、`model`）
4. 显示关键聚合指标：总行数、总成本、总 token、各 model 行数与成本 Top 榜
5. 页面显示首屏渲染耗时与内存占用（`performance.memory`，浏览器可用时）

## 注意

- 数据以本地文件加载（fetch / FileReader 均可），页面注明加载方式与用时
- 性能数字用**真实测量**，不要造假
- 提供加载失败时的错误提示

## 输出

单个 HTML 文件，浏览器直接打开即可运行。
