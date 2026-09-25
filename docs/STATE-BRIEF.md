# STATE-BRIEF — 恢复指挥速览（供上下文压缩后整份粘贴）

> ★ 本文件供参谋侧（对话侧 AI）恢复用；执行侧不得据此行动，执行侧看 RUNNER-HANDOFF.md。
> 用途：Claude 上下文被压缩后，用户把本文件整份粘贴，即可恢复指挥所需最小信息。
> 更新时间：**2026-09-23（Z 段前置存档轮）**；此前为 2026-09-16（R 轮）。
> ★★ 只需读本文件的「Z0 当前状态」一节即可恢复到当前状态，不必翻全部 SESSION-LOG。

## Z0 当前状态（2026-09-23 21:35，Z 段前置存档）
- **当前停点：Z-1（等用户视觉核对）** —— tick 究竟是「世界时间」还是「帧计数」需要一次视觉核对
  （看画面上单位是否在移动、年份是否在走），据此定夺 D-033 是否作废。
  Z-1 由用户执行，可与建档并行；**存档完成前不得执行 Z-2 及其后步骤**。
- **当前阻塞项：B-21R** —— 暂停无法长时间保持，根因未明（两次事件：①推定 13:40:24 解冻；
  ②用户 20:30~21:00 手动暂停后世界仍在跑）。两次均无执行侧动作、均无 GUI 痕迹可用。
- **当前未决假说（均未排除）**：
  - **S-1.1 情形 Q**（最高优先）：九次采样 paused 字段九次皆 True 而 tick 以 58.27/s 增长；
    D-033「tick 为唯一运行判据」从未被独立验证（系由 B-18 那次矛盾顺手定下）。
    若 tick 实为帧计数（59.06/s ≈ 60 FPS 常见帧率），**D-033 整条作废**，且此前全部
    「世界在跑」结论（ENV-1、C-3、B-21R、三次王国基线作废的归因）都需重算。
  - **S-1.2 S3 挂起**：13:30 pause 成立（睡眠之前）；18:03:07 唤醒之后历次冻结全部失败（含手动暂停）；
    PID 5464 自 2026-09-21 19:30 运行至今（CPU 累计 55,912 s）其间经历 S3
    ⇒ 推测 S3 挂起/恢复破坏了 Unity 侧时间或暂停状态机。
  - **S-1.3 冲突**：S-1.2 要求解冻在唤醒之后；C-3 残差重算指向解冻在睡眠之前（约 13:40:24）。
    两者不能同时成立，当前都不排除；Z-1 与 Z-2 可同时对两者取证。
- **当前判据集状态（S-1.4）**：`tick` 存疑（待 Z-1）；`paused 字段` 已知不可信（B-18）；
  `autosave` 存疑（若冻结成立仍按 301 s 出新槽则为实时驱动，须移出判据集，Z-3d）；
  `语义量（年份/population/单位移动）` 尚未建立（Z-1 起纳入）
  ⇒ **当前项目没有一个已验证的「世界是否在运行」的可靠判据。**
- **已放行的并行工作（Z-5 解耦）**：Y-4 代码修复与 Y-5 fixture 不依赖冻结，即刻可做；
  fixture 用**非冻结态实采原文**，每份标注采集墙钟与 tick，文件头注明
  「非冻结态采样，仅供解析逻辑回归，不得用作世界状态基准」；
  **Y-7 联机段整体推迟至冻结成功之后**，不得在运行态造墙（building_count 会被自然建造污染）。
- **ENV-1 措辞修正（2026-09-23）**：删除「API pause 可靠」「已实测可长时间保持」等措辞，改为待定；
  机制层判断保留（pause/resume/set_speed 是 0.4.0 原生节奏命令，与手动暂停等效，QQBOT-DESIGN 5.2）。
- **本状态全文出处**：`docs/SESSION-LOG/2026-09-22-R4-V.md` 的
  【C-3 残差复核】【Y-3.2/Y-3.2b 采样】【Z 段前置 · 未决假说与裁决】三节；
  执行侧交接见 `docs/RUNNER-HANDOFF.md`（§1 当前任务书与停点、§5 缺陷表含 B-21R）。
- **存档缺口（2026-09-24 01:25 更新）**：S-1.5(d) 的「Z 段任务书全文（Z-1~Z-6 + 停报项 l/m）」
  **已到达并逐字落盘**（见 `docs/SESSION-LOG/2026-09-22-R4-V.md` 同节的「Z 段任务书全文（原文照录）」）；
  S-3 的「Z-1 / Z-6 用户回答」：**Z-6 已到达**（用户原话见下 S-1.6 条）；**Z-1 仍为占位**（未到达）。
- ★★ **Z 段停点与预授权撤销（2026-09-24 01:25 立，23:5x 前移）**：当前任务书 = **Z 段**，
  停点 = **Z-0.12**（Z-0 续已完成：用户口述落盘 / Z 任务书全文照录 / 环境取证 / 账本只读核对 /
  D-037 定向备份 / 文档与镜像同步）；**W 段 B-3 造墙预授权即日起撤销** ——
  任何 `.造` / `.打` / `invoke_power` 类动作须参谋侧逐次放行。
  **下一步 = Z-1（由用户执行视觉基准核对）**：情形 P 屏幕在动 → 进 Z-2；情形 Q 屏幕静止而
  tick 仍约 59/s 增长 → D-033 整条作废、按 Z-1b 用语义量重建判据并停报等裁决；
  **Z-2 及其后步骤在 Z-1 判定前不得执行**。
- ★★★ **Z-7 / Z-8 收尾（2026-09-25 11:13~11:52）：D-041 生效、S-1.6 关闭、游戏已关**
  - **Z-7.0**（只读）：游戏/bot 均未运行；`LastBootUpTime = 2026-09-25 09:25:26`
    （★较上一记录 09-24 21:38:40 **已换过一次开机**，与 ENV-2 吻合）；save10 新规则核对
    **THREE_FILES=PASS + MAP_META_RULE=PASS**；Z-4 登记**未触发**；0x7B 出处 = **本文件第 156 行**
    （B5 第 2 条，正样本 `load_world` 命中 1）。
  - **Z-7.1**：`LAUNCH_AT=2026-09-25 11:13:20.566`、**PID 24212**（StartTime 11:13:20）、
    两桥 `/health` 11:13:46 均 200；用户经 GUI 载入 save10，原话「1、非暂停」「2、1x」「3、11：25：02」。
  - **Z-7.2 API 正路径 = 通过**：`population_alive` **1018 → 1045 → 1073**；
    各城 population S0→S1 **7 城变**、S1→S2 **6 城变**；`population_lifetime` 948→948→1052；
    国名「清泉 国」→「清泉 直隶」；`kingdoms_alive` 3→3→3、`ever_created` 15→15→15 不变。
    **`building_count` 本轮无数据**（任务书误以为 list_cities_ex 含该字段，实际在 8723 `list_cities`
    ⇒ 参谋侧担责、不补测）。
  - **年份正路径 = 通过**（用户原话：「我自己精准测试了下，一年的时间在1x下是在50s左右误差不超过2s」）
    ⇒ 人眼通道间隔定为 **≥60 s**。
  - **Z-7.4 整段取消**（M2）：本轮**零改世界动作**（未发 pause/resume）；**S-1.6 = 关闭**
    （用户原话「最小化窗口你完全可以问我，不影响的」+ 其原唯一反证基于已作废的 tick 判据）。
  - **M3**：版本核查不做（用户原话「版本号确实是我记错了」），钉死仍为 **0.51.2**。
  - **Z-7.5**：`CloseMainWindow` 成功（CLOSE_REQ_AT 11:48:44.426、MainWindowHandle 789530、
    返回值 True、+5 s 退出、8723/8724 监听 0）；退出后 save10 新规则复核 **PASS**（未存盘）。
  - **新增 D-041（世界运行判据）**：人眼 = 间隔 ≥60 s 两次读年份（1x 下约 50 s/年，其他档另测）；
    API = 300 s 窗口内 `population_alive` 或任一城 `population` 有变化即运行，全不变 = 冻结；
    `building_count` 待验证（取自 `list_cities`）；**tick / paused / autosave 不参与判定**（D-040）。
  - **Z-8 审计**（只读、只分类）：六项重算清单（ENV-1、C-3、B-21R、S-1.2、S-1.3、
    「三次王国基线作废」）**无一项被判「推翻」**；维持 = ENV-1 的睡眠事实、C-3 的取证事实、
    三次基线作废的观测事实；不可判定 = ENV-1 的 13:40:24 解冻推断、C-3 的 tick 残差结论、
    B-21R、S-1.2、S-1.3、三次基线作废中的「连续运行 12 小时」时长推断。
    详见 `docs/SESSION-LOG/2026-09-22-R4-V.md`【Z-8 审计】节。
  - **未启动 bot；未进 Z-5 / Y-3.x。**
- ★★ **（历史）Z-1 判定结果：情形 P（2026-09-25 00:29；其含义已按 Z-3.0a 修订）**：
  - 游戏由执行侧启动（`Start-Process E:\game\worldbox\worldbox.exe`，PID 11784，00:19:00.652；
    两桥 /health 00:19:26 均 200）；用户经 GUI 载入 **save10**（执行侧未调用任何载入类 API）。
  - 用户四问原文：「1、在移动；」「2、在变化；」「3、什么意思，这里面的按钮只有暂停和非暂停状态，
    目前是非暂停，速度1x」「4、789年，会变，持续上涨」。
  - 桥侧三次采样（00:28:16.364 / 00:28:26.384 / 00:28:36.401）：tick 35535 → 36128 → 36722，
    均 `paused=false`，**DIFF 593/594 per 10 s ⇒ 约 59.3 tick/s**。
  - ⇒ 屏幕在动 + 语义量（年份）在涨 + 界面非暂停，三者一致 ⇒ **情形 P：tick 为真，世界确在运行**；
    **不是情形 Q**。**Z-2/Z-3 未执行，须等参谋侧放行。**
    ★修订（2026-09-25，Z-3.0a）：**情形 P 仅证明运行态下画面与 tick 同步增长，不区分 S-1.1
    （世界时间 vs 帧计数），D-033 仍待定**。
    【已作废·仅留追溯】原表述：「情形 P：tick 为真，世界确在运行」。
  - ★ Z-2 状态（Z-3.0b）：**未执行（参谋侧裁决）**——干净进程目的已由重启后 PID 11784 替代；
    手动存盘会覆盖已核实的 save10 基准存档；**不构成合规 Z-2 执行记录**。
  - ★★ **Z-3 续停报（2026-09-25 00:40）：停点 = Z-3.1，停报条件 u 触发** ——
  Z-3.1a PASS（游戏仍 PID 11784、自 LastBootUpTime 起无睡眠/唤醒事件，正样本近 3 日
  Kernel-Power=30 条）；Z-3.1c PASS（**get_world_state 不返回年份/世界时间字段**，
  字段仅 width/height/seed/tick/paused/population_alive/population_lifetime/kingdoms_alive/
  kingdoms_ever_created/cities_alive/cities_ever_created）；
  **Z-3.1b FAIL**：save10 的 `map.meta` SHA256（386116dae9f54c39…）≠ Z-0.11 备份
  （53d7cc430d903934…）——该文件于 **2026-09-25 00:20:44**（载入世界后约 1.7 分钟）被改写，
  差异**仅两处**：`modsActive` 数组顺序 与 `timestamp`（1789990500.96 → 1790266844.79），
  **mapStats 逐键无差异**，且 `map.wbox` / `map_stats.s3db` / `EmpireCraftModData.json`
  mtime 仍为 2026-09-17 13:28:49 **未变** ⇒ 属「载入时元数据刷新」而非存盘覆盖世界。
  ⇒ **Z-3.2~Z-3.6 未执行、本轮零改世界动作（未发 pause/resume）、未启动 bot、未进 Z-4/Z-5。**
  待参谋侧裁决：① 是否接受 map.meta 的元数据刷新为可忽略差异并放行 Z-3.2~Z-3.6；
  ② 或改为以「世界数据三件（map.wbox / map_stats.s3db / EmpireCraftModData.json）」为核对集。
- ★★★ **Z-3 观测结果（2026-09-25 00:46:50~01:07:44）：S-1.1 成立 ⇒ D-033 整条作废（待裁决）**
  - **动作**（放行动作 2/2 已用尽）：1 次 `pause`（00:46:50.711，响应
    `{"ok":true,"result":{"previous_paused":false,"paused":true},"tick":100380}`）+ 1 次 `resume`
    （01:07:14.229，响应 `{"ok":true,"result":{"previous_paused":true,"paused":false},"tick":172589}`）。
  - **tick 侧**：pause 后全程 `paused` 字段=True，tick 仍以 **≈59.1 tick/s** 增长
    （Z-3b 9×15 s：101331→108454，INC=7123；Z-3c 12×60 s：112012→151016，INC=39004；
    两阶段 STEPS(+1步数) 均为 **0** ⇒ 既非甲亦非乙 ⇒ 形态落丙；pause→结束 +50,636/857 s）。
  - **语义量/视觉侧**：四个视觉检查点（T+0 / T+2.5 / T+6 / T+12）用户均答
    **①「否」（单位不动）②「813」（年份逐字不变）③「暂停状态」**；
    Z-1b 快照 A（01:01:40.329，tick=152933）与 B（01:06:45.861，tick=170906）间隔 305.5 s，
    **8 城 population/building_count/kingdom_id、王国数 2、kingdoms_ever_created 16、
    population_alive 1192 全部逐字相同**，而 tick **+17,973（≥15,000）**
    ⇒ 正命中 Z-1b.3 ⇒ **S-1.1 成立：tick 是帧计数，不是世界时间**。
  - **判据集变更（本次结论）**：①`tick` **移出**（S-1.1）；②`paused` 字段早已移出（B-18）；
    ③`autosave` **移出**（本次实测：世界冻结期间仍按 301 s 节拍出新槽 00:46:24→00:51:25→00:56:25，
    ⇒ 实时驱动）；④可用的语义量：**年份（界面读）/ population_alive / 各城 population 与
    building_count / 王国数与 ever_created / 单位是否移动**。
  - **待裁决**：D-033 存废；判据集按上述重建；并**重估此前所有「世界在跑/没跑」结论**
    （ENV-1 机器睡眠归因、C-3 残差重算、B-21R、以及三次王国基线作废的归因）。
  - **未执行**：Y-3.3~Y-3.7（P3 覆盖条款）；Z-4 / Z-5；未启动 bot；未存盘 save10。
  - 停报项：**l) 成立**（tick 非世界时间）；m)/u)/v)/w) 未触发；Z-3.1b 经参谋侧改判 PASS。
- ★★★ **Z-3 收尾（2026-09-25 01:18）：D-033 已作废、D-040 生效、游戏已 graceful 关闭**
  - **Q1（Z-3.6 用户原话逐字）**：「我这边看恢复画面运动了」。
  - **Q2 裁决**：`D-033` 标【已作废 · 见 D-040】（保留不删）；**新增 D-040**：
    「tick 为帧计数（约 59/s），不是世界时间，不得作运行判据；autosave 为实时驱动，移出判据集」
    （依据 = Z-3 全程观测，本地 `e644394…` / 镜像 `35c4513…`）。
    **判据集（临时）**：年份 / population / building_count / 王国数 = 负路径已验证、正路径待验证；
    单位移动 = 仅人眼；`paused` 字段暂不恢复为判据（B-18 不变）。
    **待重算清单（只登记不重算）**：ENV-1、C-3、B-21R、S-1.2、S-1.3、「三次王国基线作废」的归因；
    附注 **B-21R 可能源于 tick 判据误判（未验证）**。
  - **Q3 关机**：`CloseMainWindow`（未强杀、未存盘）——CLOSE_REQ_AT=2026-09-25 01:15:57.634、
    返回值 True、+5 s 存活 False、EXITED_WITHIN_60S=True、8723_LISTENING=0。
    退出后 save10 按**新核对规则**复核：**THREE_FILES=PASS**（三件 SHA256 逐字一致）+
    **MAP_META_RULE=PASS**（差异仅 modsActive 与 timestamp；mapStats 无差异）；各文件 mtime 未变。
  - **Q4**：停点 = **Z-3 收尾**；未启动 bot；未进 Z-4 / Z-5 / Y-3.x；本轮补丁零改世界动作。
- ★ 绑定登记（Z-3.0c，只登记不修改）：bindings 唯一一条绑定指向 **kingdom 17**，该王国实测**已不存在**；
  待 defect E + D-036 通道实装后处理，本轮不动库。
  - 附带实测（供 S-1.1 继续取证，不改本轮判定）：速度 **1x** 而 tick 仍 ~59.3/s（与"tick≈60 FPS
    帧计数"相容）；**tick 载入后由 ~6.73e6 重置为 10,235 ⇒ 不随存档持久**；
    **seed 字段本次读到 2，此前 V-3/W/X/Y 各段一律 6，但城市名与坐标逐座一致 ⇒ seed 跨会话会变、
    不可作世界身份判据**；save10 载入后 estado：8 城全为"县"、kingdom 17 **不存在**、city 6 属 kingdom 14、
    四国 = 7 胡勒目斯国 / 14 清泉国 / 15 苍穹国 / 16 白鹿公国；用户读到的年份 789 ↔ 存档声明 784（+5 年）。
- ★★ **意外重启（2026-09-24）与两条新增条目**：
  - 事件：意外关机 **21:28:33**（EventLog Id=6008），21:29 重启，21:38 正常关机后再重启
    （LastBootUpTime 2026-09-24 21:38:40）；游戏与 bot 均**非优雅终止**、重启前未手动存盘；
    autosave 最后持久化点 21:28:16（世界回退量 ≈17 秒）。**本次重启不构成合规 Z-2 执行记录**
    （对照组来自非优雅终止：进程干净但存盘完整性未验证）。
  - **ENV-2 每日必经关机或 S3**（环境约束；用户原话：「我去睡觉的话，电脑要么关机要么就是开
    睡眠的，不能整晚开」）⇒ 任何跨夜进程**不可假定连续**；佐证：09-24 两次 S3
    （6:55:23→12:19:54、14:10:41→18:26:49）。
  - **S-1.6 窗口最小化/失焦影响暂停状态**（新增候选假说，**未验证，仅登记**）：
    出处 = Z-6 用户回答「按 gui 的按钮一次的，有切窗口（暂停后就最小化做别的去了）」。
  - 账本现状（Z-0.10 只读）：`pending_approval=0`、`points=0`、`ledger_log=0`、无孤儿冻结；
    `bindings=1`（kingdom_id="17"、state=active）。

## B1 环境栈（一行式）
WorldBox 0.51.2 学习版 E:\game\worldbox；NML 1.2.0.1+补丁 v1.1（StreamingAssets\mods\）；
EmpireCraft 0.4.2（Mods\EmpireCraft\，NML 现场 Roslyn 编译；许可证存疑：
  MIT 版权行为 WorldBoxOpenMods / GameBanana 标 CC BY-NC-ND 4.0，仅私用不分发）；
BepInEx 5.4.23.5 + WorldBoxBridge 0.4.0（127.0.0.1:8723，28 命令）；
自写插件 WBAIBridge 0.3.0（wbai.bridge，127.0.0.1:8724，/health + POST /cmd：list_armies、list_cities_ex）；
worldbox-mcp==0.4.0（venv src\mcp-server，29 工具）；基线 Assembly-CSharp SHA256 51D275F0…69F。

## B2 四类产品可行性（各一行）
攻击=可行（invoke_power 打击类 A 档，6/6 accepted 5/6 有效）；
防御=部分可行（墙类 A 档且持久，塔类 B 档已停测）；
资源花费=可行（CityResources get/change/set 全 public，可读写）；
调兵=部分可行（参照 EmpireCraft 前线架构，行为层介入，D-017，尚未实现）。

## B3 已落地能力 与 已否决路线（各 ≤5）
已落地：WBAIBridge 插件与 0.4.0 并存通过（双端口/401 生效/编译无损）；
  spawn 未验证：Q 主任务书从未下发，旧表述系文档幻觉（R0 已定性）；EmpireCraft 前线调兵架构研读完毕（4B）。
已否决：回合制+截图发战报（D-010，改投屏 RTS）；MultiplayerBox 联机（D-002）；
  agents.json 运维台 token 沿用（D-014 未启用）；clone_rain/equipment_rain 增兵（B 档，Q 轮弃）。

## B4 进度
进行到 Q-MEM 轮（文档固化收尾）；步骤 0-4 全开（token 轮换、spawn 补测、只读资源命令、
  收工文档）；下一步按 Q 轮剩余待办推进（QQ bot 设计稿 R 轮填写）。

## B5 已知未解决风险（≤5）
1. 城市归属可经 list_cities_ex 精确获取；但 warrior_slots/max_warriors 滞后
   （暂停态不重算）、is_alive 不反映人口（建筑未全毁恒 true）；
2. load_world 对本机存档格式已知缺陷（0x7B SimpleZip），定性不修；
3. 塔类 power 无法经桥接放置（B 档，已停测）；
4. 调兵尚未实现（D-017 待定反射复用 vs 自实现）；
5. 三套注入共存稳定但 NML 三监听器构造失败等已知问题未修。

## B6 关键接口口径备忘
paint_tile 吃 tile id（list_tiles，如 mountains），非 power id（tile_mountains 报 UNKNOWN_ASSET）；
isOkToSendArmy 阈值 = 0.7f；query_actors 返回 name/asset_id/kingdom_id/x/y；
capital_id 可用于定位 City.storages（Building.resources 聚合读数）。

## B7 文档索引（仅路径，字节数只记在 SESSION-LOG）
- docs\DECISIONS.md
- docs\MEMORY.md
- docs\AGENT-CONTRACT.md
- docs\FEASIBILITY-SELFHOSTED-COMMANDS.md
- docs\STATE-BRIEF.md
- docs\RUNNER-HANDOFF.md
- baselines\assets-vanilla-empirecraft\powers.txt
- docs\DESIGN-BRIEF-ADDENDUM.md
- docs\QQBOT-V1-ARCH.md
- docs\SESSION-LOG\（各轮记录）；docs\QQBOT-DESIGN.md（设计空白稿）