# Model Battle Demo

抖音模型测评系列的测试文件仓库。

## 简介

本仓库用于存储抖音「模型 Battle」系列视频中涉及的测试文件、代码示例和相关资源。

## 目录结构

```
model-battle-demo/
├── model-battle-ep07-demo/    # 第7期：Boids 算法测试
│   ├── boids-buggy.html       # 有 bug 的版本
│   ├── boids-fixed.html       # 修复后的版本
│   └── prompt.md              # 测试提示词
├── model-battle-ep08-demo/    # 第8期：折射玻璃
│   └── prompts/prompt.md
├── ep01-prompt.md             # 第1期：旋转六边形弹球
├── ep02-prompt.md             # 第2期：3D 圆锥滚地
├── ep03-prompt.md             # 第3期：九连环
├── ep04-prompt.md             # 第4期：Rust 10 个 Bug AI 父亲
├── ep05-prompt.md             # 第5期：mini Redis
├── ep06-prompt.md             # 第6期：SPH 粒子流体
├── ep07-prompt.md             # 第7期：3D Boids Debug
├── ep08-prompt.md             # 第8期：折射玻璃光线追踪
├── ep09-prompt.md             # 第9期：3D 魔方求解器
├── ep10-prompt.md             # 第10期：OpenClaw 35 万 star 修真 bug
├── ep11-prompt.md             # 第11期：多足生物群仿真
├── ep12-prompt.md             # 第12期：弹珠漏斗耐力赛
├── ep13-prompt.md             # 第13期：1v1 晋级赛 Kanban 拖拽看板
├── ep14-prompt.md             # 第14期：开放式投票 XCrawl
├── ep15-prompt.md             # 第15期：8 个国产 AI 修 vue3 真实 PR
├── ep16-prompt.md             # 第16期：K2.7-Code vs K2.6 自相残杀
├── ep17-prompt.md             # 第17期：国产三巨头对决（K2.7-Code / GLM-5.2 / M3）
├── ep18-prompt.md             # 第18期：AI 元层对决（让 AI 写 AI 主题视频）
├── ep19-prompt.md             # 第19期：V4-Pro vs V4-Flash · 3 harness 双变量
├── ep20-prompt.md             # 第20期：Grok 4.5 vs GPT-5.6 vs Fable 5
├── ep21-prompt.md             # 第21期：V4 Pro vs V4 Flash · harness 对比
├── ep22-prompt.md             # 第22期：Kimi K3 vs MiniMax M3
├── ep23-prompt.md             # 第23期：Gemini 3.6 Flash vs 3.5 Flash-Lite vs Grok 4.5
├── ep24-prompt.md             # 第24期：V4 Flash vs GLM 5.2 vs V4 Pro
├── ep19-sonnet5-vs-v4pro-prompt.md  # EP19 番外：Sonnet 5 vs V4-Pro（3 题公平对决）
├── ep25-prompt.md             # 第25期：Qwen 3.8 Max vs V4 Flash vs GPT-5.6 Luna
├── ep26-prompt.md             # 第26期：可视化规则引擎 + 多人在线竞价系统
├── ep28-prompt.md             # 第28期：多米诺骨牌刚体物理仿真
└── ...
```

## 各期索引

| 期 | 主题 | 题目类型 |
|---|------|----------|
| 01 | 旋转六边形弹球 | 物理仿真（碰撞/反射） |
| 02 | 3D 圆锥滚地 | 物理仿真（无滑约束） |
| 03 | 九连环 | 算法（拓扑/递归） |
| 04 | Rust 10 个 Bug AI 父亲 | 编程修复 |
| 05 | mini Redis | 系统设计 |
| 06 | SPH 粒子流体 | 物理仿真（流体） |
| 07 | 3D Boids Debug | 算法（群智能/找 bug） |
| 08 | 折射玻璃光线追踪 | 物理仿真（光学） |
| 09 | 3D 魔方求解器 | 算法（图论/搜索） |
| 10 | OpenClaw 35 万 star 修真 bug | 编程修复 |
| 11 | 多足生物群仿真 | 物理仿真（生物动力学） |
| 12 | 弹珠漏斗耐力赛 | 物理仿真（粒子） |
| 13 | 1v1 晋级赛 Kanban 拖拽看板 | Web 应用（拖拽） |
| 14 | 开放式投票 XCrawl | 数据采集 |
| 15 | vue3 PR 修复 | 编程修复（3 档梯度） |
| 16 | K2.7-Code vs K2.6 自相残杀 | 内部对决（5 题） |
| 17 | 国产三巨头对决 | 评估框架 |
| 18 | 元层对决：AI 写 AI 主题视频 | 元层任务 |
| 19 | V4-Pro vs V4-Flash · 3 harness 双变量 | 光学渲染 + 纯文本推理 |
| 20 | Grok 4.5 vs GPT-5.6 vs Fable 5 | 编程修复（Kanban 状态一致性） |
| 21 | V4 Pro vs V4 Flash · harness 对比 | harness 变量实验（复用 19 两题） |
| 22 | Kimi K3 vs MiniMax M3 | 3D 场景复刻 + 魂系 Boss 游戏 |
| 23 | Gemini 3.6 Flash vs 3.5 Flash-Lite vs Grok 4.5 | 3D 场景复刻 + 在线代码编辑器 |
| 24 | V4 Flash vs GLM 5.2 vs V4 Pro | 数据可视化 + SQLite 知识库 |
| 19番外 | Sonnet 5 vs V4-Pro | 光学渲染 + 迷宫算法 + 复利数学 |
| 25 | Qwen 3.8 Max vs V4 Flash vs GPT-5.6 Luna | BI 数据大屏 + 百万行性能挑战 |
| 26 | 可视化规则引擎 + 多人在线竞价系统 | 业务流程 + 实时状态/并发边界 |
| 28 | 多米诺骨牌刚体物理仿真 | 真实碰撞物理 |

## 使用说明

每个 episode 文件夹对应一期视频的测试内容，可直接在浏览器中打开 HTML 文件查看效果。

## 作者

GuLu9527 - 抖音模型测评系列
