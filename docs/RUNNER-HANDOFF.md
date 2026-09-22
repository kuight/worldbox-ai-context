# RUNNER-HANDOFF — 执行侧换对话恢复用

> ★ 本文件供执行侧（本地 Agent）换新对话恢复用；参谋侧（对话侧 AI）不据此行动。
> 参谋侧看 STATE-BRIEF.md；技术结论在 MEMORY.md，本文件不写技术结论。

## 1. 当前任务书编号与进行到哪一步
- 当前任务书：**K-2（放行执行；含 K-1 三项修正与新增 K-1.5）**，上游为 J-1（B-10 只读取证）与 R4-2i（装配与协议对齐）。
- 已落地裁决：D-031（point_start=30 接线，DECISIONS.md 已入档）；镜像每轮同步（AGENT-CONTRACT 常设条款）；
  B-10 解冻范围仅 resolver.py；defect E 与 B-11 只登记不修；**E 修好前禁止任何重绑（含 .bind 16）**。
- 本轮已完成：K-0（D-031 + 镜像条款独立提交 **2e720dbe5d538990ecb9f8a074a5999092f29b07**）；
  K-1.5（R4-2i 业务改动基线提交 **c50f0fc9ded03ffb8d9b42ca585f4823ca0ed4ad**，147 tests）；
  K-2（B-10 修复 + B-13 + K-2.8 双键 + K-2.9 拆名 + K-2.6b 区分 detail，仅 resolver.py）；
  K-3（点数三键接线 + 装配期 fail-fast，仅 bot.py）；
  K-4（出站/入站留痕日志，仅 bot.py）；
  K-5（真桥信封原文 fixture 2362/2140/1321 B + 12 条新用例，**pytest 159 passed 连跑 3 次全绿**）；
  K-6（G1-G6 全过）；K-7.1~K-7.3（pending_approval=0 红线确认、F 轮方式优雅停、新入口重启核验全过）。
- **本轮停点：K-7.4 —— 等用户在白名单群发一次 `.国情`**（回执原文从 K-4 新增日志取，不截图转写）；
  其后 K-7.5~K-7.7（对照表 / bindings 逐字不变 / points 行数）→ K-8 收尾（HANDOFF/MEMORY/SESSION-LOG/提交/镜像/完成汇报）→ 停报等裁决是否进 V-3。
- 当前 src 侧 commit：**c50f0fc9ded03ffb8d9b42ca585f4823ca0ed4ad**（K-1.5 基线）+ K-2/K-3/K-4/K-5 未提交改动（K-8.3 收口），pytest **159 用例**全过。

## 2. 本轮已动过的文件
- **src\qqbot\resolver.py**（K-2：read_situation 改"取全部匹配行"，治 B-10 的四个表象：城数恒 1 /
  人口取首城 / 出兵线取首城 / 城名显示国名；新增 K-2.8 双键 is_alive + is_alive_raw、
  K-2.9 country 与城名拆开（禁止回落城名）、K-2.6b 两种 0 城情形的 detail 可区分（数字 id vs 名字）；
  K-2.5 军队计数键从 kingdom_name 改为 kingdom_id 经 str() 归一化（治 B-13）。
  **:173-204 对账门逐字未动**（K-2.7；git diff hunk 从旧 278 行起）。
- **src\qqbot\bot.py**（K-3：`_point_settings(cfg)` 装配期 fail-fast + Ledger 显式接 initial/cap/accrual_interval_sec，
  治 B-9「配置 30 未接线，实际生效 5」，D-031 落地；K-4：`_send_group` 出站回执留痕
  `logger.info("OUT at=%s group=%s text=%s", ...)` + `_route` 入站留痕
  `logger.info("IN at=%s group=%s qq=<QQ len=%d> cmd=%s", ...)`（B-12），不动 adapter.py。
  此前 R4-2i 轮：serve() 生命周期（治缺陷 A）、`_validate_bridge_config` 装配期 fail-fast、
  `_link_error_text` 文案分叉（B-6）、`_ACTION_CAPABILITY` 改 `invoke_power`（B-8）、
  read_population 改走 get_world_state（B-5）、executor tokens 注入。
- **src\qqbot\tests\test_situation.py**（新，K-5：12 条用例，fixture 从 HTTP 边界喂真桥原文）。
- **src\qqbot\tests\bridge_fixtures.py**（新，K-5.1：三份真桥信封原文字节，`repr(bytes)` 生成，字节级精确；
  由 `E:\work\r4v-scaffold\_k5_gen_fixture.py` 从实时桥抓取）。
- src\qqbot\tests\test_resolver.py（军队桩字段改 kingdom_id，配合 K-2.5 的 id 匹配）；
  src\qqbot\tests\test_bot.py / test_protocol.py（cfg 补 point_start/point_cap/point_rate_minutes 三键）。
- docs\DECISIONS.md（D-031）与 docs\AGENT-CONTRACT.md（镜像同步节奏）——已独立提交 2e720db；
  docs\RUNNER-HANDOFF.md（本文件，K-8.1 重写）。
- **（沿用）bot 进程 PID 的真相 —— 查/杀 bot 不要信 Start-Process 的返回值**：
  `.venv-qqbot\Scripts\python.exe` 是 **redirector**，真正持有 6199 监听的是它 fork 出的基础解释器子进程
  ⇒ 判定「bot 的 PID」一律**以 netstat 上「谁持有 6199 监听」为准**。
- 脚手架（仓库外 E:\work\r4v-scaffold\，只挪不删）：`_r4v_launch.retired.py`（旧入口已退役）、
  `_r4v_stopbot.py`（v2：AttachConsole + SetConsoleCtrlHandler(NULL,TRUE) 自保 + CTRL_C_EVENT(0)，配合
  `_r4v_startbot.py` 的 CREATE_NEW_CONSOLE 独占控制台，治缺陷 F）、
  `_r4v_stopbot.v1-shared-console-broken.py`（v1 归档）、`_r4v_console_probe.py`（GetConsoleProcessList 取证）、
  `_j1_forensic.py`（J-1 只读取证）、`_k5_gen_fixture.py`（K-5 fixture 生成）。

## 3. 已过闸门 / 停在哪道闸门
- 已过（R4-2i 全段）：I-0 勘查；I-1 真桥协议探针（协议表定稿：8723 用 `name`、8724 用 `command`，
  两桥 token 头均 `X-WB-Token`，**8724 的 /cmd 无 ok 字段、8723 有**；体积 2362/2140/1321 B，
  余量 27.75×/30.62×/49.61× ⇒ B-4 不登记）；I-2 实现（serve 生命周期治缺陷 A；协议表按显式 bridge id；
  装配期 fail-fast；错误映射表定稿）；I-3 测试（147 passed）；I-4 闸门 G1-G6；I-5 重启与入口切换
  （缺陷 F 根因取证 + 方案甲修复 + 两轮启停验证）；I-6 缺陷登记；I-7 未走（被 J-1 取代）。
- 已过（J-1）：B-10 根因坐实（resolver.py:278 next(...) 取首条 + :297 只塞一条 ⇒ 城数恒 1 /
  人口 49 / 出兵线 11/11 / 城名显示国名；对账门无罪——cache_size=8 与 count=8 对账通过）。
- 已过（K 轮）：K-0 / K-1.5 / K-1 三项补证（broadcaster 以 is_alive 过滤与 §5.5 字面冲突，
  但 resolver 输出 is_alive=True 即可规避）→ 用户放行 → K-2/K-3/K-4/K-5/K-6 全过 → K-7.1~K-7.3 全过。
- **停在哪：K-7.4**（用户发 `.国情`）→ K-7.5~K-7.7 → K-8 收尾 → 停报等裁决是否进 V-3。

## 4. 待贴原始输出（尚未落盘到 SESSION-LOG 的原文）
- K-7.4 的 `.国情` 回执原文（**从 K-4 新增日志取**：src\qqbot\_r4v_bot_run_z.err.log 的
  `OUT at=... text=...` 行）+ 同刻三份桥原文 → K-8.2 的 SESSION-LOG 本轮文件。
- J-1 三份真桥原文（8724 list_cities_ex 2362 B / 8724 list_armies 2140 B / 8723 list_cities 1321 B）：
  已逐字节落为 src\qqbot\tests\bridge_fixtures.py（K-5.1），取证脚本 E:\work\r4v-scaffold\_j1_forensic.py。
- R4-V / J-1 阶段回执：<USER-DIR>\完成汇报.txt
  （K-8.5 将先另存为 完成汇报-R4V-J1.txt 再覆盖写 K 轮回执）。
- 上一轮 R4-2i 回执保留：<USER-DIR>\完成汇报.pre-R4V-R4-2i.txt（52,875 字节）。

## 5. 缺陷登记（汇总；R4-V 联机段起累计）
- **缺陷 A（已修，结案）— executor.start() 从未被调用 ⇒ 桥接调用永久挂起**
  build_services 只构造 Executor（bot.py 原 :149），全仓无 async 生命周期入口；任何走桥接的指令
  （.国情/打/造/批准/read_population）永久挂在 item.future 上（零回复零日志 bot 存活）。
  修复：bot.serve()（校验 → 装配 → start → adapter.run → finally stop）+ `python -m qqbot` 正式入口；
  TEST1（未 start）6 秒超时复现、修复后两轮启停验证 worker 正常 + finally 日志。
- **缺陷 B（已修，三层）— executor 协议与两桥真实形状不匹配**
  层1：请求不带 X-WB-Token（两桥 401）；层2：请求键名错（8723 要 `name`、8724 要 `command`，实测 400 BAD_ARGS）；
  层3：executor.py 的 ok 判定要求 2xx 必含 ok 布尔，而 8724 的 /cmd 成功体**无 ok 字段** ⇒ 每次成功被判 LINK_ERROR。
  修复：BRIDGE_PROTOCOLS 协议表（显式 bridge id，不嗅端口）+ token 进请求头 + 按桥声明 ok 语义
  （8723 声明含 ok，缺 ok=协议违约；8724 声明不含 ok，以 result 存在为成功）+ 信封归一化（result→data.*）。
- **缺陷 B-3（已修）— resolver 读军队/城市的 country 字段，真桥为 kingdom_name**
  真桥条目内没有 country 键 ⇒ 恒空串。修复：军队归属读 kingdom_name；K-2.5 起军队计数键改为 kingdom_id。
- **缺陷 B-4（不登记）— BODY_CAP_BYTES 潜在静默截断**
  实测 list_cities_ex 2362 B（余量 27.75×），约 295 B/城 ⇒ >≈22 城余量跌破 10×，>≈222 城真超限。
  超限路径已改显式报错（含实际字节数 + 日志），常量维持 65536。
- **缺陷 B-5（已修）— read_population 命令与路径双错 + 漏 await**
  原实现拿 list_cities 读 data.population_alive（该命令响应里没有此字段），且漏写 await；
  真路径 = 8723 get_world_state 的 population_alive。已修并注明口径警示：
  ★ 全世界 population_alive 与王国人口是两个口径，禁止互换。
- **缺陷 B-6（已实现，语义不确定性保留）— 500 GAME_CRASH 的回执文案分叉**
  实测 invoke_power 缺 x/y → 500 GAME_CRASH（System.ArgumentException），调用方参数错可能以 5xx 出现；
  世界是否已动不可知 ⇒ 文案「桥接异常，已退还 N 点（世界状态未确认）」，不含「未送达」。
- **缺陷 B-7（登记不修）— ok_to_send 无消费者、无 C2 守卫**
  `ok_to_send` 在 src\qqbot（含 tests）0 命中（正样本 max_warriors 命中自验通过）；
  army 265（count_units=1 / max_warriors=0 / ok_to_send=true）的除零恒真无人拦。
- **缺陷 B-8（已修）— _ACTION_CAPABILITY="invoke" 实测 404 UNKNOWN_COMMAND**
  真名 invoke_power；不修则每次攻击/建造都被拒。
- **缺陷 B-9（已修；表述更正）— point_start 未接线，实际生效 5**
  准确表述：config point_start=30 / point_cap=30 / point_rate_minutes=10，而 bot.py 原 :168-170 只传 db_path，
  ledger.py:111-113 默认 initial=5 / cap=30 / accrual_interval_sec=600.0 —— cap 与 accrual 与配置巧合一致，
  只有 initial 真错。修复：K-3 显式接线 + 装配期 fail-fast（缺失/非整数/<=0 即抛，禁止回落默认值）。
- **缺陷 B-10（已修，本轮）— read_situation 把"首城"当成"整个王国"**
  根因：resolver.py 原 :295-296 注释自述的「list_cities_ex 为每国一条」假设错误（真桥每城一条，
  J-1.2-a count=8、kingdom 17 占 6 条），:278 next(...) 取首条 + :297 只塞一条 ⇒
  四个表象：城数恒 1、人口取首城（49）、出兵线取首城（11/11）、城名显示国名（苍穹 帝国）。
  对账门无罪（cache_size=8 与 8723 count=8 对账通过；缺陷在门之后的行选择）。
  修复：取全部匹配行 + 全国求和 + 真实城名 + K-2.8 双键（is_alive 恒 True 供 broadcaster /
  is_alive_raw 仅记录）+ K-2.9 country 与城名拆开 + K-2.6b 两种 0 城情形 detail 可区分。
  **C2 解除**：「未开兵役」分支在 kingdom 17 上结构性不可达的问题已由 B-10 修复解除，
  并由 K-5.2e（由真实原文派生、kingdom 17 各城 max_warriors 全置 0）覆盖。
- **缺陷 B-11（登记不修）— bindings 实际列名与 ARCH §4 表结构差异**
  实际 6 列（qq_id PK INTEGER / kingdom_id TEXT / kingdom_name_snapshot TEXT / bound_at REAL /
  bound_by TEXT / ★state TEXT NOT NULL DEFAULT 'active'★）vs ARCH:79 声明 5 列 ⇒ 多出 state；
  且 kingdom_name_snapshot 实读 '17'（非国名快照）——成因：bot.py:354
  `bind_async(qq, name, name, "qq")` 把输入的 "17" 同时当 kingdom_id 与 kingdom_name_snapshot 传入。
- **缺陷 B-12（已修，本轮）— 出站回执无日志**
  adapter 不记录消息文本（stderr 仅 4 行启动日志）⇒ 回执取证只能靠用户转写。
  修复：bot.py 出站/入站留痕日志（OUT/IN 行，含墙钟/群号/qq 掩码/命令类型/回执全文）。
- **缺陷 B-13（已修，本轮）— 军队计数键用 kingdom_name 字符串相等**
  国名会漂移（实测「金夜 叛乱」→「苍穹 帝国」）⇒ 改为 kingdom_id 经 str() 归一化后相等；不设默认回落。
- **缺陷 B-14（登记不修，待用户拍板文案）— broadcaster 的 `or 0` 吃掉 resolver 刻意保留的 None**
  armies 取数失败时 resolver :305-314 返回 None（未知，绝不报 0），但 broadcaster.py:35
  `int(s.get("army_count", 0) or 0)` 把 None 吃成 0 ⇒ 回执显示「军队数：0」，与 B-2.4
  「绝不报 0 支军队」的声明矛盾。当前行为已由 K-5.2i 快照锁定（测试注释写明非期望行为）。
- **defect E（登记不修；★E 修好前禁止任何重绑，含 .bind 16★）— .bind 未校验 kingdom_id 存在性**
  全仓 list_kingdoms 仅命中 executor.py 协议表映射；bot.py:_h_bind 直接 bind_async ⇒ 未实装
  （后果：可绑定不存在的王国，如已消亡的 14 / 不存在的 16）。ARCH §4.2 要求绑定时校验。
- **缺陷 F（已修，结案）— 停机脚本误伤同控制台全部进程**
  根因：CTRL_C_EVENT 无法限定进程组（组号非零返回成功但组内收不到），传 0 = 发给调用者所在控制台
  全部进程，而 bot 与 dsh agent 共用控制台（GetConsoleProcessList 实证：dsh.exe 13692 与 node.exe 8168
  在清单内）⇒ 多次打断 dsh agent 会话。修法（方案甲）：`_r4v_startbot.py` 以 CREATE_NEW_CONSOLE
  独占控制台 + `_r4v_stopbot.py` v2（AttachConsole + SetConsoleCtrlHandler(NULL,TRUE) 自保 + CTRL_C_EVENT(0)）；
  两轮启停验证：bot 退出、6199 释放、dsh 与 pwsh 均存活、finally 日志 + stopped。
- **缺陷 3（结案）— v1 缺持续运行入口**：由 serve() + __main__.py 修复（R4-2i）。
- **缺陷 4（未实装）— ARCH §1.4「链路已恢复」播报**：grep 0 命中（正样本自验通过）；待 adapter 侧重连播报。
- **缺陷 5（沿用）— 旧库孤儿冻结 held=2 而 pending_approval=0**（关联 D-030；证据在改名留存的旧库）。
