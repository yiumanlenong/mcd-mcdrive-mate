# 麦麦路上 McDrive Mate

> **为"在路上的人"服务的麦当劳点餐决策助手**
> 何时出发 · 去哪家店 · 点什么 · 到店即取 —— 路上 30 秒对话，到店零等待。

基于[麦当劳中国官方 MCP Server](https://github.com/M-China/mcd-mcp-server) 与[腾讯 WorkBuddy 智能体](https://www.workbuddy.cn)构建的开源 Skill。

---

## 1. 项目介绍

现有的麦当劳 AI 点餐助手大多在解决"**吃什么**"（营养配比）和"**怎么省**"（优惠券精算）。但在真实生活里，最频繁的场景是：

> 你在开车、在通勤、在赶时间——**吃饭的决策必须在到达之前完成**，而且你真正在意的是时间，不是价格。

**麦麦路上 McDrive Mate** 专注这个场景，提供四件事：

| 能力 | 说明 |
|---|---|
| 🕐 场景感知点餐 | 自动识别早餐档/午市/下午茶/夜宵，只推荐当前时段真正可售的商品 |
| 🏪 选店决策卡 | 到店取 / 得来速 / 麦乐送三方案对比，**按预计总耗时排序，不比价格** |
| ⏱ 预约单杀手锏 | 按"预计到达 + 5 分钟缓冲"创建预约单（基于 MCP v1.0.4+ 预约能力），到达即取 |
| ↩️ 反悔零成本 | 一句话取消；"改单"= 取消重下，匹配"计划赶不上变化"的通勤现实 |

**与其他参赛项目的差异**：别人比"吃什么、怎么省"，本项目解决"**何时、何地、怎么拿到**"——全链路时间最优。

## 2. 目标用户

- 🚗 **自驾通勤族**：早晚高峰经过麦当劳，想在车里完成一切
- 🛣 **得来速用户**：想知道哪家店车道排队短、提前锁定菜单
- 🏃 **赶时间的人**：会议间隙/下课路上，只有 10 分钟
- 🌙 **夜班族**：深夜找还在营业的店，吃口热乎的再回家
- 👨‍👩‍👧 **接送孩子的家长**：顺路取餐，不想带着孩子在店里等

## 3. 安装方法

### 3.1 申请麦当劳 MCP Token

1. 访问麦当劳开放平台 https://open.mcd.cn/mcp
2. 手机号验证码登录 → 右上角【控制台】→【激活】
3. 同意服务协议后复制 MCP Token

### 3.2 接入腾讯 WorkBuddy（推荐，支持 WorkBuddy 专项联动）

1. 打开 WorkBuddy → 设置 → Connector（MCP）→ 添加自定义 Connector
2. 粘贴 [mcp-config.example.json](mcp-config.example.json)，将 `${MCD_MCP_TOKEN}` 替换为你的 Token
3. 新建智能体"麦麦路上 McDrive Mate"：
   - 系统提示词：粘贴 [`skills/mcdrive-mate/SKILL.md`](skills/mcdrive-mate/SKILL.md)
   - 知识文件：上传 [`skills/mcdrive-mate/prompts/`](skills/mcdrive-mate/prompts/) 下 3 个片段
   - 启用 Connector `mcd-mcp`

### 3.3 接入其他 MCP Client

Cherry Studio / Cursor / Trae / VSCode 等支持 Streamable HTTP 协议的客户端，同样粘贴配置即可。注意事项：

- Token 限流 600 次/分钟（超限返回 429）
- 券/订单/积分类工具依赖麦当劳会员账户，建议先在麦当劳 App 登录过一次
- 完整 Tool 说明见 [MCP_INTEGRATION.md](MCP_INTEGRATION.md)

## 4. 使用示例

### 示例 1：晚高峰预约单

```text
👤 18:25 我到XX路附近，帮我安排晚饭，到店拿
🤖 现在 17:40，晚市正餐档。为你找到 3 家门店：
   ✅ XX路餐厅 · 到店取（预约 18:30，含 5 分钟缓冲）· 预计总耗时 ~5 分钟
   备选：YY大道餐厅 · 得来速 · ~9 分钟｜麦乐送 · ~35 分钟
👤 就这家，老样子
🤖 你的老样子是麦辣鸡腿堡套餐(零度)×1，已用本店券 -¥3。
   应付 ¥xx.x，预约取餐 18:30。确认下单吗？
👤 确认
🤖 已下单 ✅ 支付链接已发给你。
   改主意随时说"取消订单"。
```

### 示例 2：深夜反悔

```text
👤 刚下夜班，想吃点热的再回家
🤖 现在 23:10，附近还在营业的有 2 家（A 家 24h / B 家 24:00 关门），
   推荐 A 家得来速，预计 ~8 分钟。深夜档有麦麦鸡、香芋派…
👤 堵死了，来不及了，取消
🤖 已取消 ✅ 要不要改成 19:15 的预约单？
```

更多端到端脚本见 [docs/demo-scenarios.md](docs/demo-scenarios.md)（含通勤早餐、得来速、夜宵三个完整场景）。

## 5. 项目结构

```
├── README.md                    # 本文件
├── CONTEST_DECLARATION.md       # 参赛声明
├── LICENSE                      # MIT 开源许可
├── .gitignore                   # 挡住 .env / 真实 mcp-config.json，保护 Token
├── MCP_INTEGRATION.md           # 麦当劳 MCP Server/Tool/调用流程/业务价值
├── mcp-config.example.json      # 脱敏 MCP 配置示例
├── workbuddy.md                 # WorkBuddy 联动说明与对话上下文（专项奖励申报）
├── docs/
│   ├── architecture.md          # 决策流程图
│   ├── demo-scenarios.md        # 演示脚本
│   └── changelog.md
├── skills/mcdrive-mate/
│   ├── SKILL.md                 # Skill 主提示词（决策原则/流程/输出格式）
│   └── prompts/                 # 分场景策略片段
│       ├── time-slot.md         # 时段 → 品类心智映射
│       ├── store-pick.md        # 按总耗时排序的选店逻辑
│       └── order-flow.md        # 下单确认/预约单/取消规范
└── examples/                    # 示例对话实录
    ├── commute-breakfast.md     # 通勤早餐 · 预约单
    ├── drive-thru-pickup.md     # 深夜得来速
    └── order-cancel-rebook.md   # 反悔与改约
```

## 6. 设计原则（为什么值得用）

1. **比时间，不比价格** —— 路上场景的硬需求是确定性，不是省 2 块钱
2. **账单确认后才下单** —— AI 永远不替你付钱，`calculate-price` 出账单卡、你确认、才 `create-order`
3. **时段先行** —— 拒绝"早上推荐巨无霸"的菜单心智错位
4. **不确定就问** —— 地址、方式、时间三要素齐了才动手

## 7. Roadmap

- [x] P0：时段感知 / 选店决策卡 / 预约下单 / 账单确认
- [ ] P1：出发前速览卡（积分过期提醒 + 券有效期 + 当月活动）
- [ ] P1：常用路线记忆（"通勤路线 A 老样子"一键复购）
- [ ] P2：天气 × 路况联动出发时机建议（需第三方数据源）

## 8. FAQ

**Q1：会自动下单扣款吗？**
不会。任何 `create-order` 之前都必须经过「`calculate-price` 出账单卡 → 你明确回复确认」两步（见 [skills/mcdrive-mate/prompts/order-flow.md](skills/mcdrive-mate/prompts/order-flow.md) 的前置检查清单）。未支付订单随时可一句话取消，不留资金占用。

**Q2：我的 MCP Token 安全吗？**
Token 只保存在你本地客户端的配置里。本仓库的 `mcp-config.example.json` 仅含环境变量占位符，`.gitignore` 已挡住 `.env` 与真实配置文件入库。请勿把 Token 粘贴到任何公开位置。

**Q3：调用有限制吗？**
麦当劳官方 MCP 限流 600 次/分钟（超限返回 429）。本 Skill 通过会话内复用菜单结果、并行合并查询来控制频率，正常对话远达不到上限。

**Q4：必须用麦当劳会员账号吗？**
查门店、看菜单不需要；但券（`query-my-coupons`）、下单（`create-order`）、积分（`query-my-account`）依赖你的会员账户状态，建议先在麦当劳 App 登录过一次再使用。

**Q5：为什么"比时间不比价格"？**
路上场景的核心痛点是**不确定性**（排队多久、到店有没有），不是差价。省钱精算类项目已经解决"怎么省"，本项目专注"怎么最快拿到"——两者互补而非竞争。

## 9. 许可与声明

- 本项目为"麦当劳程序员创意开发大赛"参赛作品，参赛声明见 [CONTEST_DECLARATION.md](CONTEST_DECLARATION.md)，代码以 [LICENSE](LICENSE)（MIT）开源
- 依赖的麦当劳 MCP 服务须遵守[麦当劳 MCP 服务规则](https://cdn.mcd.cn/cms/pages/MCPServerRules.html)
- 本项目为个人非商业开源项目，与麦当劳官方无隶属关系；"麦当劳""McDonald's"及相关商标权利归麦当劳所有
