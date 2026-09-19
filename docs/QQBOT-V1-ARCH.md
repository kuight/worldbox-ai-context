# QQBOT-V1-ARCH — QQ bot v1 架构稿（2026-09-17）
> 受众：参谋侧 + 用户审阅。实装以任务书为准。
> 依据：QQBOT-DESIGN 五~八节（R3-V 定稿）、DESIGN-BRIEF-ADDENDUM C1-C6 / D1-D4。
> 范围：纯 Python，不需用户开机即可开发；真机联调才需要开游戏。

## 一、进程拓扑
QQ 群 ←→ NapCat（QQ 客户端侧，OneBot v11 实现）
        ↕ WebSocket 双工（默认反向：bot 作 WS 服务端监听 127.0.0.1:6199/ws，D-024）
      bot.py（单进程 asyncio）
        ↓ HTTP
      8723 worldbox-mcp 0.4.0（28 命令）
      8724 WBAIBridge 0.3.0（/health、/cmd: list_armies / list_cities_ex）
        ↓
      ledger.sqlite3（绑定表 + 行动点 + 冷却 + 待批 + 审计）

1.1 连接方式：双模式适配（D-024），默认反向 WS（bot 作 WS 服务端）。
    默认（反向）：bot 监听 127.0.0.1:6199，路径 /ws；NapCat 侧 websocketClients
    （反向 WS 客户端）enable=true、指向 ws://localhost:6199/ws。
    依据 D-024 与本机 NapCat 现状——当前配置正是「反向 WS 客户端
    ws://localhost:6199/ws」，并无 websocketServers 项，故 v1 默认反向
    = 复用现配置、零改动。
    ★ messagePostFormat="array" 是硬要求：需要按段解析 @ 与文本，
      string 格式会把 CQ 码混进正文。
    ★ reportSelfMessage=false 防止 bot 自己的回执被当成新指令（自激环）。
    反向模式由 bot 监听端口；NapCat 断链按 reconnectInterval=30000 最长 30 秒
    重连，开发期需容忍（D-024 影响面）。
    可选（正向）：config 切 forward 后由 NapCat 作 WS 服务端（websocketServers
    加项、端口 3001）、bot 作客户端，断线指数退避重连（1→2→4→8→30 秒封顶），
    heartInterval=30000。
1.2 双工单连接同时承载「事件上报」与「API 调用」，发消息不需要另开 HTTP 通路。
    API 调用带 echo 字段做请求-响应配对（OneBot v11 标准）。
1.3 若将来换 NoneBot2，仍用反向 WS（NapCat websocketClients →
    ws://127.0.0.1:8080/onebot/v11/ws，NoneBot 侧需 ReverseDriver）。
    该切换只影响 adapter 层，parser 以下不变。

【原 §1.1 正向 WS 表述 — 已作废（D-024）；保留作考古线索，处理方式同 DESIGN-BRIEF】
    NapCat 侧：websocketServers 加一项，enable=true、host=127.0.0.1、port=3001、
    token 必填、messagePostFormat="array"、reportSelfMessage=false、heartInterval=30000。
    bot 侧：websockets 客户端，断线指数退避重连（1→2→4→8→30 秒封顶）。
    ★ bot 不监听任何端口，与 8723/8724/6099 无冲突面。
1.4 心跳 30 秒无到达视为链路死亡 → 主动重连，并在恢复后向群播一条「链路已恢复」。

## 二、模块划分（一个进程内的八个模块，单向依赖；D-029 由九改八）
adapter   ← WS 收发、echo 配对、重连；唯一接触 OneBot 协议的地方
parser    ← 群消息 → 结构化指令（文法见三节），不做任何 IO
resolver  ← 符号化目标 → 精确坐标（走 8724 list_cities_ex / list_armies；
            8723 list_cities 仅供 count 对账；query_actors 取坐标已作废）
ledger    ← 行动点账本、冻结/扣款/退款、冷却表、绑定表读写
            （D-029：binding 并入 ledger，不再单列模块）
catalog   ← 商品表：价格、冷却、是否进待批、文案
executor  ← 唯一调用 8723/8724 的地方；串行队列
receipt   ← 两段式回执与延迟结算
approval  ← 待批队列、60 秒超时
★ 所有桥接调用只走 executor；禁止其它模块直接 requests，便于统一超时与审计。

## 三、指令文法（只接符号化目标，不开放裸坐标）
3.1 触发条件：群消息且（@bot 或以前缀「.」开头）。私聊仅支持 .bind 与 .me。
3.2 指令表（v1）
    .bind <王国名或编号>      绑定自己的国（一人一国，重绑需 God 批）
    .me                       查自己：国、点数、冷却
    .国情                     我国：城数、总人口、军队数与出兵线快照
    .战况                     全图：各国人口、交战对
    .打 <目标> [用 <法术>]     打击类，默认 lightning
    .造 <墙名|房> [在 我城]    建设类，目标只能是我国/我城
    .慢速 / .常速 / .暂停 / .继续   节奏控制（任何人可发，全群公示，不自动触发）
    .价目                     商品表与冷却规则
3.3 目标文法（<目标> 只允许这几种形态）
    我城 / 我国首都 / 我国 <城名>
    <国名>首都 / <国名> <城名> / <国名>军队
    ★ 不接受「(x,y)」这类裸坐标；解析失败一律回「看不懂这个目标」+ 示例。
3.4 歧义处理：国名/城名做前缀匹配，命中多个则回候选列表让其重发，不猜。
3.5 法术白名单（v1）：lightning、meteorite。
    earthquake / tornado 标「效果未实测」不上架；atomic_bomb / paint_tile 走待批。
3.6 文案硬要求（QQBOT-DESIGN 7.1/7.2）：
    .打 城市目标 且法术=lightning 时，执行前先回「lightning 打城市几乎无效
    （实测 -4），确认请加『确认』重发」——防买错，不是拦。

## 四、绑定表（binding）
表 bindings(qq_id PK, kingdom_id, kingdom_name_snapshot, bound_at, bound_by)
4.1 绑定即正向解析的唯一依据（反向校验坐标→归属不可行，不做）。
4.2 绑定时校验 kingdom_id 仍存在（8723 list_kingdoms）；国灭则该绑定标记 dead，
    持有者可 .bind 重绑一次，不消耗点数。
4.3 kingdom_name 只存快照做显示；一切逻辑用 kingdom_id。
4.4 God（用户 QQ 号，写死在 config）不绑国，拥有 approve/deny/撤销。

## 五、坐标解析（resolver，R3-V 后重写）
5.1 城市目标 = 8724 list_cities_ex 的 (x,y)，即 getTile().x/y，tile 网格坐标，
    与 8723 invoke_power 同坐标系（R3-V-4 实测 16/16 指向城市区域）。
    ★ 旧的 query_actors 聚类取密度中心已作废（偏差 0~56 格且无规律，R3-V-5）。
5.2 军队/统帅目标 = 8724 list_armies 的 captain_x/y，字段本身准确（R2-9b）。
    ★ C6：非暂停态必须在 invoke 前 1 秒内重取一次坐标，用最新值投点。
5.3 城市列表缓存 TTL 10 秒；军队坐标不缓存，每次现取。
5.4 C5 对账门：每次刷新城市缓存后，与 8723 list_cities 的 count 比对；
    不一致则丢弃本次结果、退避 2 秒重拉，最多 3 次，仍不一致则本轮拒绝服务
    并回「世界仍在加载，稍后再试」。理由：世界加载中会静默返回不完整列表
    （首拉 1 城、复拉 6 城，代码无缺陷）。
5.5 C4：不得用 is_alive 判断「城已毁」（isRekt 看建筑全毁，非人口归零，
    is_alive==false 至今从未观测到）。城市消失与否只以「是否还出现在列表里」为准。
    实装注记（R4-2d-fix 回归 C4）：resolver 存活判定唯一依据为「city_id 是否仍
    出现在 list_cities_ex 列表中」，is_alive 字段仅记录不参与判定；CITY_GONE
    判定前必须先过 5.4 的 C5 对账——对账未通过一律 UNSTABLE，绝不误判成
    「城已毁」（加载不全 ≠ 城消失，防误伤）。

## 六、行动点账本（ledger）
6.1 v1 用时间当货币（D4）：在线/在群成员每 10 分钟 +1 点，上限 30 点，
    新人初始 5 点。★ 费率与上限均为可调参数，写 config 不写代码。
6.2 建议初版价目（参谋定，待用户拍板）
    lightning 3 点 / 发；meteorite 12 点 + 该城 30 分钟冷却；
    墙 7 种同价 1 点 / 次；living_house 1 点 / 次。
    依据：meteorite 单发即可把 202 人城市清到 20 人，是唯一战略级删除手段
    （QQBOT-DESIGN 7.2 平衡警告），必须最贵 + 最长冷却。
6.3 冷却两类（6.2/6.4 原则）
    城市冷却：表 city_cooldown(city_id, until_ts)；打击类共用同一把冷却，
      冷却期内对同城重复打击照收全价且回执明示「预期无效」。
    个人冷却：建设类每人 60 秒一次，不吃城市冷却。
6.4 扣款事务顺序（防双花与防白扣）
    校验点数 → 冻结（hold）→ 入 executor 串行队列 → 桥接返回 accepted
    → 落账（commit）；桥接拒绝/超时/异常 → 解冻全额退款。
    ★ 「accepted 但无效果」不退款（B 档与冷却期内无效属规则内结果，
      回执写清楚；见 QQBOT-DESIGN 8.6）。
6.5 单一 asyncio 写者 + SQLite 事务；executor 串行本身也保证桥接侧不并发
    （桥接主线程串行 + 30 秒超时）。
6.6 表 ledger_log(id, ts, qq_id, action, cost, state, bridge_resp_digest)
    全量审计；★ token 永不入库、不入日志、不回显。

## 七、两段式回执（receipt，C1 硬约束）
7.1 伤害延迟结算，立即采样必然报 0（MEMORY §24-1），故打击类必须两段：
    T0   读 population_alive 存基线（8723）
    T0+  invoke_power → 立即回「已受理：<目标> <法术>，已扣 N 点，落地结算中」
    T0+8s 再读 population_alive → delta → 补发「战果：人口 -X」
    ★ 8 秒取值来自 R2-9 观测的 5~10 秒结算窗口，写 config 可调。
7.2 delta 归因声明：世界同时在自然波动，单发 delta 只作参考，
    回执文案写「本段时间内人口变化」，不写「你杀了 X 人」。
7.3 建设类无延迟问题，单段回执即可（墙类 A 档、accepted 即生效且持久）。
7.4 失败分三类，文案必须区分：桥接拒绝（已退款）/ 游戏接受但无效果（不退款）
    / 链路异常（已退款并记 incident）。

## 八、待批队列（approval，8.4 定稿）
8.1 进队列的指令：atomic_bomb、paint_tile（v1 均未上架，接口先留）、
    以及 catalog 里标 needs_approval 的项（meteorite 视最终定价而定）。
8.2 时限 60 秒，超时默认拒绝并全额退款。
8.3 ★ 不碰世界时间：审批期间世界照常跑，绝不自动 pause。
    理由：自动暂停会让任何人连发待批指令就冻住世界；且否决权是初版脚手架，
    终局拆除时不该牵连世界时间控制。
8.4 God 在群里回「批 <编号>」/「否 <编号>」；编号 4 位随机，避免误批。
8.5 队列容量 5，满则直接拒绝新申请（不排长队）。
8.6 终局替代：改「高价 + 长冷却 + 全群公示」，无需人审。

## 九、播报（broadcaster）
9.1 .国情 / .战况 的出兵线展示必须先判 max_warriors == 0（C2）：
    为 0 时浮点除零 = +Infinity，isOkToSendArmy 恒 true，
    不判零就会系统性说谎（R3-V-7 实机确认：adults≤15 的四城全中）。
    文案：分母为 0 时显示「未开兵役（成年人口不足）」，不显示比值。
9.2 warrior_slots / max_warriors 标注为「上一周期快照」（C3；实证 id=3 城
    population 202→20、adults=11 而 warrior_slots 仍挂 112），不做实时判断。
9.3 0.7f 出兵线只读展示，不作为任何玩法杠杆（D-020）。
9.4 v1 不做自动定时播报，只响应指令；大规模交战时可发一条「建议切慢速」
    的建议消息（不自动改速度，5.2）。

## 十、目录与配置
src\qqbot\
  bot.py            入口，装配九个模块
  adapter.py        WS 收发 + echo 配对 + 重连
  parser.py         文法（纯函数，可单测）
  binding.py  ledger.py  catalog.py  resolver.py  executor.py
  receipt.py  approval.py
  config.example.toml   ★ 真 config 不入库（含 NapCat token 与桥接 token）
  tests\            parser / ledger / 文案 的纯单测，不需要游戏
config 键：napcat_ws_url、napcat_token、god_qq、group_whitelist、
  bridge_8723_url、bridge_8723_token、bridge_8724_url、bridge_8724_token、
  point_rate_minutes、point_cap、settle_delay_sec、approval_timeout_sec、prices{}
★ group_whitelist 必填：bot 只在白名单群响应，防串群。

## 十一、v1 验收清单（分两段，不需要一次开机）
离线段（不开游戏）：parser 文法用例全过；ledger 冻结/退款/双花用例全过；
  adapter 对着一个假 OneBot WS 服务端跑通收发与重连；
  executor 对着桩 HTTP 跑通超时与退款路径。
联机段（开游戏 + NapCat）：.bind → .国情 → .造 墙 → .打 城市（两段回执）
  → 冷却期重打（明示无效且扣款）→ 待批流程一次批一次否一次超时
  → 断 NapCat 重连 → 断桥接退款。

## 十二、待决（需用户拍板）
Q1 接入方式 —— 已拍板，见 D-024（默认反向 WS，bot 监听 6199/ws）。
Q2 费率与价目 —— 已拍板，见 D-027。
Q3 meteorite 是否进待批队列 —— 已拍板，见 D-026（进待批）。
Q4 group_whitelist 与 god_qq 的实际值 —— 待定（R4-V 联机段前给值）。
