# EP26 Prompt

## Q1：可视化规则引擎

请在当前目录实现一个可运行的浏览器端可视化规则引擎。

## 目标

用户通过节点编排一条自动处理流程，并能够运行、查看每个节点的输入输出和执行结果。重点是业务逻辑真实可运行，不是只做静态展示。

## 必须支持

1. 节点类型：开始、条件分支、数据变换、审批、通知、结束。
2. 节点连接、删除、复制、重新连接。
3. 条件分支至少支持金额比较、字段匹配和 AND/OR 组合。
4. 流程运行时显示当前节点、成功、失败、跳过状态。
5. 每个节点可查看输入、输出、错误原因和执行耗时。
6. 至少提供三条预置流程：订单审核、退款审批、告警升级。
7. 支持输入一份 JSON 数据运行流程。
8. 支持保存、加载和重置流程。
9. 支持运行历史，能够查看每次运行的最终状态和事件日志。
10. 在没有后端和外部 API 的情况下，使用本地确定性模拟完成审批和通知。

## 公开验收场景

- 订单金额大于 1000 时进入人工审批，否则直接通过。
- 审批拒绝时必须进入结束节点，并记录拒绝原因。
- 缺少金额字段、金额为负数、条件无法判断时，流程必须明确失败，不能静默通过。
- 同一流程连续运行两次时，第二次不能继承第一次的节点状态、输入或输出。
- 重置后回到初始流程和初始数据。

## 实现约束

- 使用项目现有技术栈，不依赖外部 API。
- 项目必须能通过一个明确的启动命令运行，并在本地 HTTP 服务中打开。
- 在 README 或项目说明中写出安装、启动和测试步骤。
- 提供至少一个可直接打开的 demo 页面。
- 所有交互必须真实生效，禁止使用仅用于装饰的假按钮。
- 保持代码结构清晰，避免把全部逻辑压缩成不可维护的单文件。

## 交付

完成后请说明：启动命令、主要文件、已实现功能、已知限制和建议的手工验收路径。不要伪造测试结果。

---

## Q2：多人在线竞价系统

请在当前目录实现一个可运行的浏览器端多人在线竞价系统。重点是实时状态、并发冲突、余额约束和边界条件，不是只做静态 UI。

## 必须支持

- 商品列表、商品详情、拍卖状态和倒计时；
- 当前最高价、最低下一次出价、出价历史；
- 多用户模拟出价；
- 用户余额校验；
- 自动出价和最高预算；
- 最后几秒延时拍卖；
- 流拍、成交和结果导出；
- 刷新后状态恢复；
- 明确显示成功、拒绝及失败原因。

使用本地确定性数据和模拟服务，不依赖外部 API。金额使用整数最小货币单位，禁止使用浮点数表示余额和成交金额。

## 测试对接

应用必须支持 `test` 模式，并暴露 `window.auctionTest` 或等价桥接对象。测试脚本只通过外部行为验收，不依赖你的框架、目录或内部函数名。

```ts
interface AuctionTestBridge {
  reset(seed?: string): Promise<void>;
  advanceTime(milliseconds: number): Promise<void>;
  listAuctions(): Promise<AuctionSummary[]>;
  getAuction(id: string): Promise<AuctionState>;
  placeBid(input: { auctionId: string; userId: string; amount: number }): Promise<BidResult>;
  setAutoBid(input: { auctionId: string; userId: string; maxAmount: number }): Promise<BidResult>;
  settle(auctionId: string): Promise<SettlementResult>;
  exportHistory(auctionId: string): Promise<BidRecord[]>;
}
```

`AuctionState` 至少包含：`status`、`currentPrice`、`highestBidderId`、`minimumNextBid`、`endsAt`、`extensionCount`、`bidHistory`、用户余额和自动出价状态。每个操作返回机器可读的状态和 `reason`。同价出价必须按可复现规则处理。`advanceTime()` 是测试时间唯一推进方式，测试不得真实等待倒计时。

## 交付

提供明确的启动命令、README、可操作 demo、测试模式说明和已知限制。不要伪造测试结果。完成后说明主要文件和手工验收路径。
