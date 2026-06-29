# 国产 AI 编程大乱斗 EP10 · 国产开源 × Claude Sonnet · OpenClaw 真实 PR 三连击

> **使用说明**：本文档是通用提示词，7 个参赛模型都收到一样的内容，但每个模型运行在独立的工作目录里，互不影响。

---

# 角色

你是全球 AI 编程能力评测的一名参赛模型。本次评测目标：评估你修复真实开源项目 bug 的能力。

# 工作区

你被分配了一个独立的工作目录（`projects/model-battle-ep10/workspaces/demo-X/`，X 是分发者告诉你的 1–7）。

该目录是 [OpenClaw](https://github.com/openclaw/openclaw)（35 万 stars，Peter Steinberger 用 TypeScript 写的 AI agent）的**独立本地 clone**，当前 HEAD 为 commit `111b65a6fb`（branch `ep10-base`），是 3 个目标 PR 合并前的公共 base。

**你和其他 6 个模型完全隔离**，互不影响。在本目录里你拥有完整的 git 仓库。

- 包管理：pnpm。**不要跑 `pnpm install`** —— 依赖不用装，你只需要修改源码。
- 测试框架：vitest。**不要跑测试或构建** —— 后期由审核员统一验证。
- 源码位于 `src/`，可以自由 `cat`、`grep`、`ls` 任何文件。

# 任务

修复下面 3 个 OpenClaw issue。每题独立一个分支，从 `ep10-base` (`111b65a6fb`) 拉出：

| 题 | 难度 | 分支名 |
|----|------|--------|
| Q1 | ⭐⭐ 易 | `fix/q1` |
| Q2 | ⭐⭐⭐ 中 | `fix/q2` |
| Q3 | ⭐⭐⭐⭐ 难 | `fix/q3` |

完成顺序自由。**严格按以下工作流执行**：

```bash
# 0. 确认在你分配的工作目录（每题开始前都检查一次！）
pwd   # 输出应包含 workspaces/demo-X

# 1. 确认 working tree 干净
git status   # 必须 "nothing to commit, working tree clean"

# 2. 从 base 拉新分支
git checkout -b fix/qX ep10-base

# 3. 改文件（只动 src/，根据 issue 描述定位）

# 4. 提交
git add -A
git commit -m "fix(qX): <一句话总结>"

# 5. 切回 base，准备下一题
git checkout ep10-base
```

**警告**：
- 每题必须 commit，不能留未提交改动
- 切回 ep10-base 前 `git status` 必须 clean
- 三道题分支名严格是 `fix/q1`、`fix/q2`、`fix/q3`
- 不要 `pnpm install`、不要跑测试、不要 build

# 输出要求

- **不要**把 diff 粘到 chat。你的答案就是 commit 本身。
- 每题完成后输出一句话状态（如 "Q1 完成，分支 fix/q1"）
- 全部完成后输出 `git log fix/q1 fix/q2 fix/q3 --oneline -3 --all`

# 修复要求

- ✅ **最小修改**：只改必要文件
- ✅ **零新依赖**：不要 `pnpm add`
- ✅ **风格一致**：遵循已有文件的命名/import/类型标注
- ✅ **必要时补回归测试**：如果原文件有对应 `.test.ts`
- ❌ **不要**改无关文件（package.json / tsconfig.json / CHANGELOG.md 除非题目明确需要）
- ❌ **不要**重写整个文件，只 patch 关键行

# 评分维度（你看得见，但评分由审核员做）

| 维度 | 权重 |
|------|:---:|
| 能跑（git apply / pnpm build 不报错） | 25 |
| 过测试（跑对应 .test.ts 能 pass） | 30 |
| 修复正确（与官方 PR 语义等价） | 25 |
| 改动质量（最小+不引入新依赖+风格一致） | 10 |
| 测试补充 | 10 |

---

# 题目一 · ⭐⭐ 易 · Cron Slack Delivery Bug

## Issue Body

### Summary

After upgrading to 2026.5.12, isolated cron jobs with `delivery.mode: announce, channel: slack` fail with `Unsupported channel: slack`, even though the Slack plugin is installed, configured, enabled, and successfully serving the main session (socket-mode connected, `slack_post` working).

The agent run itself succeeds — only the framework-level `delivery.announce` step fails, so the cron is silently broken: the work completes but the report never reaches the channel.

### Root cause

The 5.12 release notes call out:

> Plugins: externalize Slack, OpenShell sandbox, and Anthropic Vertex so their runtime dependency cones install only when those plugins are installed.

After externalization, `@openclaw/slack`'s `openclaw.plugin.json` ships with:

```json
"activation": {
  "onStartup": false
}
```

Combined with two existing code paths, this becomes a silent regression for isolated cron deliveries:

1. **`channel-bootstrap.runtime.ts` requires opt-in to lazy-load a non-startup plugin.** `resolveOutboundChannelPlugin` only calls `bootstrapOutboundChannelPlugin` when `params.allowBootstrap === true`.

2. **The cron isolated-agent delivery path does NOT pass `allowBootstrap: true`.** In `dist/jobs-*.js`:
   ```js
   async function resolveOutboundTargetWithRuntime(params) {
     try {
       const loaded = tryResolveLoadedOutboundTarget(params);
       if (loaded) return loaded;
       const { resolveOutboundTarget } = await loadTargetsRuntime();
       return resolveOutboundTarget(params);  // ← no allowBootstrap
     } ...
   }
   ```
   And in `dist/targets-*.js`:
   ```js
   function resolveOutboundTarget(params) {
     return resolveOutboundTargetWithPlugin({
       plugin: resolveOutboundChannelPlugin({
         channel: params.channel,
         cfg: params.cfg,
         // ← allowBootstrap omitted
       }),
       ...
     }) ?? { ok: false, error: new Error(`Unsupported channel: ${params.channel}`) };
   }
   ```

3. **`delivery-queue` does pass it** (`dist/delivery-queue-*.js` line 403 and 490: `allowBootstrap: true`), which is why main-session deliveries and tools like `slack_post` keep working — but cron's isolated-agent delivery path doesn't.

### Suggested fix

`resolveOutboundTarget` in `src/infra/outbound/targets.ts` should pass `allowBootstrap: true` when called from a runtime context (cron isolated, agent-delivery). The simplest version: just thread `allowBootstrap: true` through the `resolveOutboundChannelPlugin` call inside `resolveOutboundTarget`.

## 关键源码线索（src/ 下实际文件）

```
src/cron/isolated-agent/delivery-target.ts      ← cron 调用入口
src/cron/isolated-agent/delivery-target.test.ts ← 现有测试
src/infra/outbound/targets.ts                   ← resolveOutboundTarget 真身
src/infra/outbound/targets.test.ts              ← 测试
src/infra/outbound/channel-resolution.ts        ← resolveOutboundChannelPlugin
src/infra/outbound/channel-bootstrap.runtime.ts ← bootstrapOutboundChannelPlugin
```

注意：issue 引用的是 `dist/` 路径（构建产物），实际改的是 `src/` 下的 TypeScript 文件。

---

# 题目二 · ⭐⭐⭐ 中 · MCP Plugin-Tools AbortSignal 转发

## Issue Body

### Bug type

Behavior bug (incorrect output/state without crash)

### Summary

The standalone MCP plugin-tools server (`createToolsMcpServer` + `createPluginToolsMcpHandlers`) discards the SDK's per-request `RequestHandlerExtra.signal`, so an in-flight `tool.execute` keeps running after the host sends `notifications/cancelled` or after the stdio transport closes.

### Expected behavior

The probe tool's `execute` receives a defined `AbortSignal` and observes `aborted === true` after the parent's `ctrl.abort()`, mirroring the SDK contract documented in `@modelcontextprotocol/sdk` `RequestHandlerExtra.signal: AbortSignal` and the optional third argument on `AnyAgentTool.execute(toolCallId, params, signal?)` (`src/agents/tools/common.ts:22-28`).

### Actual behavior

The probe tool's `execute` is invoked with `signal === undefined`. A wire-level run records:

```text
executeInvoked:      1
signalDefined:       false
signalIsAbortSignal: false
abortObserved:       false
signalAbortedAtEnd:  false
```

The client observes the AbortError but the server-side `tool.execute` runs to completion with no cancellation channel.

### Root cause

- `src/mcp/tools-stdio-server.ts:17` wires the request handler as `async (request) => handlers.callTool(request.params)`, **discarding the SDK's second argument `extra: RequestHandlerExtra`** (which carries `signal: AbortSignal`).
- `src/mcp/plugin-tools-handlers.ts:45` then calls `tool.execute(\`mcp-${Date.now()}\`, params.arguments ?? {})` **without a signal**, so the optional 4th parameter of `tool.execute(toolCallId, params, signal?)` is permanently `undefined`.

### Files involved

```
src/mcp/tools-stdio-server.ts:16-19
src/mcp/plugin-tools-handlers.ts:45-70
src/agents/tools/common.ts:22-28  (AnyAgentTool.execute signature with optional signal)
```

### Additional information

- The handler API itself also lacks a `signal` parameter (`callTool: async (params: CallPluginToolParams)` at `plugin-tools-handlers.ts:45`), so even if the SDK wrapper threaded `extra.signal` through, there would be no channel into `tool.execute`. **Both pieces are missing.**
- Same wiring is shared by `plugin-tools-serve` and `openclaw-tools-serve` (both go through `createToolsMcpServer` → `connectToolsMcpServerToStdio`). The fix is one wrapping signature change plus a 1-line parameter addition on `callTool`, plus passing `signal` as the 4th argument to `tool.execute`. **Patch is ~3 hunks across 2 files.**
- 你需要新建一个回归测试文件，建议命名 `src/mcp/plugin-tools-handlers.cancel.test.ts`

---

# 题目三 · ⭐⭐⭐⭐ 难 · Gateway Web Search Provider Validation

## Issue Body

### Description

Gateway fails to start with crash loop when `tools.web.search.provider` is set to `"brave"` — even though the brave plugin is correctly installed, enabled, and loads successfully in non-gateway contexts.

The gateway's pre-startup config validation checks provider availability **before** external plugins are loaded. Since Brave was externalized from core in 2026.5.x, it's never available at validation time, causing an immediate startup failure.

### Reproduction

1. Fresh upgrade from 2026.4.22 to 2026.5.12
2. Config has `"tools": { "web": { "search": { "provider": "brave" } } }`
3. Install brave plugin: `openclaw plugins install clawhub:@openclaw/brave-plugin`
4. Verify plugin is installed and enabled: shows `status: "loaded"`, `webSearchProviderIds: ["brave"]`
5. `openclaw config validate` — passes
6. Start gateway → crash loop with: `tools.web.search.provider: web_search provider is not available: brave`

### Expected behavior

Gateway should either:
1. Load external plugins before validating `tools.web.search.provider`, or
2. Defer provider validation to after plugin registration, or
3. Degrade gracefully (disable web search with a warning) rather than aborting startup

### Workaround

Change `tools.web.search.provider` to `"duckduckgo"` (a bundled provider).

### Related comment from prior issue

> "an installed/stale but unavailable web_search provider should not prevent basic gateway stop/repair/config-migration flows. A narrow repair path would be to make unavailable optional web_search providers degrade or self-disable rather than abort startup."

## 关键源码线索

```
src/config/validation.ts                          ← web_search provider validation 核心位置
src/config/config.web-search-provider.test.ts     ← 现有测试，可以加新 case
src/plugins/channel-plugin-ids.test.ts            ← 相关插件 id 测试
```

## 解题方向提示（不强制）

issue 给了 3 个候选方案。需要权衡：

- **不能让所有未知 provider 都通过**：`"provider": "bravo"`（typo）必须仍然 fatal
- **要区分 "stale plugin evidence" 和 "unknown provider"**：前者降级，后者保持严格
- **要保留 Gateway 启动 / repair / config-migration 流程**

需要补充至少 2 个回归测试（一个验降级路径，一个验 typo 仍 fatal）。

---

# 最终输出

完成 3 题后，输出以下命令的结果：

```bash
git log fix/q1 fix/q2 fix/q3 --oneline -3 --all
git --no-pager diff 111b65a6fb..fix/q1 --stat
git --no-pager diff 111b65a6fb..fix/q2 --stat
git --no-pager diff 111b65a6fb..fix/q3 --stat
```

如果某题无法完成，明确说"QX 弃权"，分支可以不创建。

开始吧。
