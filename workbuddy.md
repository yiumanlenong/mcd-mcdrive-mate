# workbuddy.md — WorkBuddy 联动说明与对话上下文

> 本文件用于"麦当劳程序员创意开发大赛"**WorkBuddy 专项奖励**申报。
> 本项目（麦麦路上 McDrive Mate）真实使用**腾讯 WorkBuddy 智能体**作为运行载体，将麦当劳官方 MCP Server 接入 WorkBuddy，实现路上场景的一句话点餐。

## 1. 为什么选择 WorkBuddy 作为运行载体

1. **Connector 原生支持 Streamable HTTP MCP**：麦当劳官方 MCP Server（`https://mcp.mcd.cn`）即插即用，无需本地代理。
2. **对话式交互契合路上场景**：用户在通勤/自驾途中用自然语言（含语音输入）完成点餐，比打开 App 快。
3. **上下文记忆**：WorkBuddy 会话内记住"路线 A 的老样子"，复购一句话完成。
4. **Skill 注入**：本项目的决策规则（`skills/mcdrive-mate/SKILL.md` + 3 个 prompt 片段）作为 WorkBuddy 智能体的系统提示词与知识文件加载。

## 2. WorkBuddy 中的配置步骤

1. 打开 WorkBuddy → 设置 → Connector（MCP）→ 添加自定义 Connector。
2. 粘贴 [mcp-config.example.json](mcp-config.example.json) 内容，将 `${MCD_MCP_TOKEN}` 替换为在麦当劳开放平台（https://open.mcd.cn/mcp）申请的 MCP Token。
3. 新建智能体"麦麦路上 McDrive Mate"：
   - 系统提示词：粘贴 `skills/mcdrive-mate/SKILL.md` 全文；
   - 知识文件：上传 `skills/mcdrive-mate/prompts/` 下三个片段（time-slot / store-pick / order-flow）；
   - 启用 Connector：`mcd-mcp`。
4. 保存后即可对话使用。

## 3. WorkBuddy 真实对话实录（2026-10-09 17:15–17:21 实测）

> 以下为 2026-10-09（周五）17:15–17:21 的端到端**真实实测**记录：经 WorkBuddy 同款通道（Streamable HTTP JSON-RPC，`initialize → tools/call`）直连麦当劳官方 MCP Server `https://mcp.mcd.cn`（Bearer Token 鉴权），所有门店、菜单、券、金额均为服务端真实返回，金额已由"分"换算为"元"。
> **全程未支付**：场景 1 创建的订单保持"待支付"并在场景 3 中取消，零资金成本。

### 场景 1：晚高峰预约单（核心场景，17:15–17:18）

```text
用户：18:25 我到云上××府附近，帮我安排晚饭，到店拿
McDrive Mate：[now-time-info] 现在周五 17:15，晚市正餐档。
  [query-nearby-stores(beType=1, searchType=2, 深圳·福海街道)]
  附近 5 家门店全部营业中、全部支持预约，今日夜市预约时段 17:14–21:45。按总耗时排：
  | ✅ 云上××府餐厅（452m，07:00–23:59）| 到店取 · 预约 18:30 | 预计 ~5 分钟，到店即取 |
  |    福永桥头得来速（473m，06:00–02:00）| 得来速车道        | 需排队 4–6 分钟（估） |
  18:30 = 你说的 18:25 到达 + 5 分钟缓冲，堵车也不慌。
用户：就这家，老样子
McDrive Mate：[order-list] 你最近一单是 10-04 的"三文鱼双虾至尊月堡四件套 +
  人气经典随心配"（¥53.4）。
  但 [query-meals(1421113, 预约 18:30)] 里月堡已下架——不推菜单里没有的东西。
  替代方案用你在这家店的常点标签商品：
  · 人气经典随心配（¥14.9，菜单标记"我的常点"）＝双层吉士汉堡＋可乐麦炫酷
  另 [query-store-coupons] 查到 3 张券，"麦旋风任选"今晚 23:59:59 到期，顺手帮你清掉：
  · 麦旋风任选1（草莓麦旋风）券后 ¥14.0 → ¥9.9
  [calculate-price]
  🧾 待确认订单 · 云上××府餐厅（桥头社区富桥大道×××号）· 到店取
  ⏰ 预约取餐 18:30（已含 5 分钟缓冲）｜外带取餐柜
  🍔 人气经典随心配 ×1  ¥14.9
  🍨 麦旋风任选1 ×1（麦旋风券）¥14.0 → ¥9.9
  ────────────
  优惠 -¥4.1 ｜ 应付 ¥24.8（原价 ¥28.9）
  确认后我马上下单。
用户：确认
McDrive Mate：[create-order] 已下单，等你扫码支付：
  订单号 1030452960****258585 ｜ 应付 ¥24.8（券抵 ¥4.1）
  支付链接 payH5Url 已生成（m.mcd.cn/mcp/scanToPay），15 分钟内支付有效
  取餐码支付后发放。改主意随时说"取消订单"，路上计划变了没关系。
  [query-order] 状态：待支付。
```

### 场景 2：深夜得来速（Drive-Thru 链路实测，17:18）

> 链路于 17:18 实测（beType=5 全链路真实调用）；"深夜"语义由门店字段支撑：塘尾 / 进场路得来速为 24 小时车道（00:00–23:59），福永桥头车道营业至 23:59。

```text
用户：刚下夜班，想吃点热的再回家
McDrive Mate：[query-nearby-stores(beType=5)] 福海街道周边 5 家得来速，都在营业：
  | ✅ 福永桥头得来速（473m，beCode 142030441）| 车道至 23:59，最近最顺路 |
  |    塘尾得来速（1.5km）/ 进场路得来速（5.4km）| 24h 车道，再晚也有   |
  [query-meals(1420304, beType=5)] 车道菜单在售热乎的：
  麦香鸡中套餐 ¥28、麦乐鸡套餐 ¥25、板烧鸡腿堡三件套 ¥34.5、派 ¥8.5……
用户：麦香鸡套餐，加个派
McDrive Mate：[calculate-price] 麦香鸡中套餐 ¥28 + 派 ¥8.5 = ¥36.5
  （现有 3 张券均不适用此组合，如实计价不打折）。
  得来速订单需报 beCode 下单，确认后 create-order，开到车道报取餐码即走。
  （本场景实录验证到计价层，未重复创建第二笔订单）
```

### 场景 3：反悔取消（17:20，取消场景 1 的待支付订单）

```text
用户：堵死了，来不及了，取消
McDrive Mate：[cancel-order(orderId=1030452960****258585, reason=1 改主意了)]
  已取消订单 ✅（未支付订单取消，零资金成本）
  [query-order] 状态：订单已取消（status=60）。
  要吃的话随时说一声，重新走一遍下单流程很快。
```

### 实测数据小结

| 项 | 值 |
|---|---|
| 实测时间 | 2026-10-09 17:15:40 – 17:20:51（GMT+8） |
| 调用工具（9 次业务调用） | now-time-info / query-nearby-stores(×2) / order-list / query-meals(×2) / query-store-coupons / calculate-price(×2) / create-order / cancel-order / query-order(×2) |
| Server 在册工具数 | 35 个（tools/list 实测） |
| 场景 1 订单 | 1030452960****258585，¥24.8，预约 18:30 → 已取消（status=60） |
| 资金动作 | 无（未支付，取消闭环） |

## 4. 申报信息汇总

| 项 | 内容 |
|---|---|
| 项目名称 | 麦麦路上 McDrive Mate |
| 参赛者 GitHub 账号 | yiumanlenong |
| 使用的智能体平台 | 腾讯 WorkBuddy |
| 接入的 MCP Server | 麦当劳官方 MCP Server（https://mcp.mcd.cn） |
| 使用的工作台能力 | Connector（MCP）、知识文件、自定义智能体提示词 |
| 对话上下文 | 见 §3（2026-10-09 17:15–17:21 真实实测，未支付闭环） |
