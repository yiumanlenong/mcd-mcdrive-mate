# MCP_INTEGRATION.md — 麦麦路上 McDrive Mate 的麦当劳 MCP 集成说明

本文档说明本项目实际使用的麦当劳 MCP Server、Tool 清单、核心调用流程与业务价值。

## 1. 使用的 MCP Server

| 项 | 值 |
|---|---|
| Server | 麦当劳中国官方 MCP Server（`mcd-mcp`） |
| 接入地址 | `https://mcp.mcd.cn` |
| 传输协议 | Streamable HTTP |
| 认证方式 | `Authorization: Bearer ${MCD_MCP_TOKEN}`（Token 从 https://open.mcd.cn/mcp 控制台申请） |
| 限流 | 600 次/分钟（项目内通过对话上下文复用查询结果避免重复调用） |
| 官方文档 | https://github.com/M-China/mcd-mcp-server |

脱敏配置见 [mcp-config.example.json](mcp-config.example.json)。

## 2. 实际使用的 Tool 清单（共 16 个）

### 2.1 时间与活动感知

| Tool | 用途 | 在项目中的角色 |
|---|---|---|
| `now-time-info` | 获取当前完整时间 | **每轮对话第一步**，驱动时段 → 品类映射（早餐档/下午茶/夜宵） |
| `campaign-calendar` | 查询当月营销活动日历 | 出发前速览卡、活动档期商品推荐 |

### 2.2 选店决策

| Tool | 用途 | 在项目中的角色 |
|---|---|---|
| `query-nearby-stores` | 查询地址附近可用门店 | 到店取/得来速方案候选集 |
| `delivery-query-addresses` | 获取用户可配送地址列表 | 外送方案前置，同时其返回信息用于外送计价前置 |
| `delivery-query-stores` | 查询地址附近可配送门店 | 麦乐送方案候选集 |

### 2.3 选品

| Tool | 用途 | 在项目中的角色 |
|---|---|---|
| `query-meals` | 查询门店当前可售菜单 | 时段过滤后的可推商品全集 |
| `query-meal-detail` | 查询餐品详情/套餐组成 | 用户问"套餐里有什么/能换什么"时 |

### 2.4 优惠与计价

| Tool | 用途 | 在项目中的角色 |
|---|---|---|
| `query-store-coupons` | 查询当前门店可用券 | 下单前最优券组合计算 |
| `query-my-coupons` | 查询用户账户优惠券 | 同上，与门店券合并去重 |
| `auto-bind-coupons` | 一键领取麦麦省全部可用券 | 首次询问后的可选动作 |
| `calculate-price` | 计算含优惠的商品/配送/应付总价 | **下单强制前置**，产出账单卡 |

### 2.5 订单生命周期

| Tool | 用途 | 在项目中的角色 |
|---|---|---|
| `create-order` | 创建订单（支持到店取/得来速/外送/预约） | 核心交付动作，含 v1.0.4 起支持的预约单与得来速场景 |
| `query-order` | 查询订单状态 | "到哪了/好了吗"→ 出餐中/已可取/已完成 三态回复 |
| `cancel-order` | 取消订单 | 路上计划有变时的反悔通道；"改单"= 取消后重下 |
| `order-list` | 查询历史订单 | "老样子"复购：提取最近一次订单商品直接进入账单确认 |

### 2.6 账户增值（P1 速览卡）

| Tool | 用途 | 在项目中的角色 |
|---|---|---|
| `query-my-account` | 查询积分/即将过期积分 | 速览卡：提醒过期积分 |
| `delivery-create-address` | 新增配送地址 | 用户无已存地址时的外送兜底 |

> 说明：`list-nutrition-foods` 仅在用户主动问热量时顺带回答，不作为项目核心功能（营养赛道不在本项目范围内）。

## 3. 核心调用流程

### 流程 A：选址决策

```
now-time-info
  └→ query-nearby-stores(用户位置/目的地)
       └→ delivery-query-addresses ── 有地址 → delivery-query-stores
            └→ [选店决策卡：按预计总耗时排序，最多 3 案]
```

### 流程 B：选品 → 计价 → 预约下单

```
query-meals(选定门店) ──(时段过滤)──→ 用户选品
  └→ query-meal-detail（可选，查套餐组成）
  └→ query-store-coupons + query-my-coupons → 最优券组合
       └→ calculate-price(商品+券+就餐方式+预约时间)
            └→ [账单卡] ── 用户明确确认 ──→ create-order
                 └→ query-order（跟踪）/ cancel-order（反悔）
```

完整交互规则见 [skills/mcdrive-mate/SKILL.md](skills/mcdrive-mate/SKILL.md)。

## 4. 业务价值

1. **把"点麦当劳"从 5 分钟 App 操作压缩为路上 30 秒对话**：时段感知 + 老样子复购，让熟客三句话完成下单。
2. **预约单消灭到店等待**：基于 MCP v1.0.4+ 的预约与得来速能力，按"预计到达 + 5 分钟缓冲"下预约单，把等餐时间挪到路上。
3. **三方案比"总耗时"而非价格**：到店取/得来速/麦乐送给出同口径的时间对比，解决路上场景真正的痛点——不确定性。
4. **反悔零成本**：取消一句话完成，匹配"计划赶不上变化"的通勤现实。
5. **券不浪费**：下单前自动完成门店券+账户券合并试算，账单透明，顺带提醒即将过期积分。

## 5. 使用前提

1. 在 https://open.mcd.cn/mcp 申请 MCP Token（手机号登录 → 控制台 → 激活）。
2. MCP Client 需支持 Streamable HTTP 协议（WorkBuddy / Cherry Studio / Cursor 均已验证支持）。
3. 部分工具（券、订单、积分类）依赖用户麦当劳会员账户状态；未登录/无账户时相关工具会返回空或错误，Skill 会引导用户先在麦当劳 App 完成账户初始化。
