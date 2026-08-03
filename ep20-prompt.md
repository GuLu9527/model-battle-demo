# EP20 Prompt — Grok 4.5 vs GPT-5.6 Sol Ultra vs Claude Fable 5

## Q1 · Agentic Kanban Repair

你要修复一个已经接近上线的 Kanban 状态管理模块。

## 工作区硬限制

你只能在当前模型自己的工作区里工作：

```text
workspaces/<your-model>/
```

当前目录里应该只有：

```text
prompt.md
seed/
```

**禁止读取或引用任何其他模型的目录、记录或答案**，包括但不限于：

```text
../../runs/
../grok-4.5/
../gpt-5.6-sol/
../claude-fable-5/
```

不要打开别人的 `record.md`、`final-kanban.py`、`test-output.txt`、`final-answer.md`。这是一场同题独立测试，读取其他模型产物视为违规。

## 任务

请只在当前工作区的 `seed/` 目录中修改代码，让所有测试通过，并补充至少 3 个有意义的测试。

运行测试：

```bash
python3 run_tests.py
```

## 限制

- 不要重写整个项目。
- 不要删除现有公开 API。
- 不要引入第三方依赖。
- 优先最小修改，保留现有数据结构。
- 修复后请在回答里说明：修了哪些 bug、补了哪些测试、还有哪些风险。

## 用户报告的 8 个 bug

1. 同列拖拽排序后，刷新页面顺序会丢失。
2. 跨列移动 task 时，原列里有时还残留同一个 task，导致重复。
3. 删除 task 后 undo，会把 task 恢复到错误列。
4. 搜索过滤后拖拽，会移动错误的 task。
5. localStorage / JSON 数据损坏时，应用直接白屏。
6. 空列拖入第一个 task 会报错。
7. done 列任务计数不会更新。
8. 移动端宽度下列不能横向滚动，最后一列看不到。

## 交付标准

- 公开测试全部通过。
- 8 个 bug 都有明确修复。
- 至少补 3 个测试。
- 代码变更尽量小。

## 重要说明

公开测试通过不代表最终满分。提交后会用隐藏测试检查边界情况，例如多步 undo、非法 index、损坏但合法 JSON schema、过滤列表跨列同名 task、done count 深层一致性等。

你看不到隐藏测试。不要针对测试名硬编码，按真实 Kanban 状态语义修。

## 错误信息来源

- 运行 `python3 run_tests.py` 会看到**断言失败详情**：每条测试带 `expected: ... / actual: ...` 对比（如 `same-column reorder must persist`），这是定位 bug 的主要错误信息。
- 初始公开测试状态为 **`RESULT 0/8 passed`**（全部失败），须自行跑测试读取失败输出再修。
- 隐藏评分器不给模型看，只做提交后评分。
- 建议跑题限制：最多 3 轮 test-fix 循环；不重写项目；不加第三方依赖；全部通过或 3 轮后停止。
