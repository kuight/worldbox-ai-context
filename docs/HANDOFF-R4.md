# HANDOFF-R4 — R4 轮在途交接（2026-09-19 记）
> 受众：参谋侧。本文件内容此前只存在于对话中，未进入 MEMORY / DECISIONS。
> 性质：R4 轮进行中的临时交接；R4-3 文档轮完成后，本文件内容应并入
>       MEMORY / DECISIONS / SESSION-LOG，届时本文件可删。
> 标签：[用户拍板] / [参谋·设计] / [执行·实测] / [联网核实]

## 0. 当前进度与在途任务（最要紧的一段）
- 已完成：R4-0（NapCat 勘查）、R4-1（venv + D-023/024/025 + 索引同步）、
  R4-2a（parser + catalog，19 用例）、R4-2b 主体（ledger，31 用例）。
- ★ 在途未完成：R4-2b 的 D-1~D-4 补丁已发给执行侧，尚未回报。内容见第 4 节。
- 未发：R4-2c（adapter + executor）、R4-2d（装配 + resolver/receipt/approval）、
  R4-3（文档同步）、R4-V（联机验收）。
  ★ R4-3 与 R4-V 的完整任务书原文此前已在对话中给出，若已丢失需重写；
    要点分别见第 7 节与第 8 节。
- 工程位置：src\qqbot\（parser.py、catalog.py、ledger.py、pytest.ini、
  __init__.py、tests\）；venv = E:\work\worldbox-ai\.venv-qqbot（Python 3.10.2）。

## 1. websockets 16.1.1 新旧 API 差异（R4-2c 必须写进任务书）
[联网核实 2026-09-19，来源：websockets 官方 upgrade 文档 stable 版]
- 新 asyncio 实现自 14.0 起为默认；旧实现移入 websockets.legacy 并已弃用，
  官方承诺维护至 2029 年 11 月。照老教程写必然跑不起来。
- 导入路径：websockets.asyncio.server.serve / websockets.asyncio.client.connect。
- 服务端 handler 只收一个参数；path 参数已移除（10.1 起无必要、13.0 起弃用），
  路径改从 connection.request.path 取。
- open / closed 属性已移除；改用 connection.state is State.OPEN，
  或直接 try/except ConnectionClosed。
- serve() 首参由 ws_handler 改名 handler（按位置传无感）。
- 客户端 async for ... in connect(...) 自带重连，但只对网络错误与 HTTP 5xx 重试，
  其余视为致命；要老行为需传 process_exception=lambda exc: exc。
- ★ 细节实装时以官方 upgrade 文档复核，勿凭本文件记忆硬写。

## 2. pytest-asyncio 1.4.0（已落实，记录理由）
默认 asyncio_mode = strict，异步测试不加标记会被跳过或报错；
asyncio_default_fixture_loop_scope 未设会刷警告。
故 src\qqbot\pytest.ini 已写 asyncio_mode = auto、loop_scope = function、testpaths = tests。

## 3. QQBOT-V1-ARCH 1.1 / 1.3 定稿替换文本（R4-3 时写入，正文当前仍是旧的正向 WS）
★ 背景：ARCH 写于 R4-0 勘查之前（正向 WS / 3001），D-024 勘查后改判为
  双模式、默认反向 WS / 6199。执行侧已两次正确指出该矛盾。替换文本如下：

1.1 连接方式 = 双模式，config 开关切换，v1 默认反向 WS（D-024，2026-09-17 改判）。
    ★ 本节原写"正向 WS / 3001"，系 R4-0 勘查前的设计；现以本文为准。
    反向模式（默认，复用本机现有配置，NapCat 侧零改动）：
      NapCat 作 WS 客户端，bot 作 WS 服务端，监听 127.0.0.1:6199，路径 /ws。
      NapCat 现有条目（R4-0 实测）：enable=true、url=ws://localhost:6199/ws、
      messagePostFormat="array"、reportSelfMessage=false、
      heartInterval=30000、reconnectInterval=30000、token 长 16。
      ★ 代价：bot 重启后最长 30 秒才被 NapCat 重连上，开发期需容忍；
        嫌慢则把 reconnectInterval 改 5000，或切正向模式。
      ★ 端口冲突风险：6199 是 AstrBot 文档中反向 WS 默认端口，
        现有条目名为 "test"，疑为旧 AstrBot 部署遗留；若重启 AstrBot 会撞。
    正向模式（备选）：NapCat 作 WS 服务端（需在 WebUI 新建 websocketServers，
      建议 127.0.0.1:3001），bot 作客户端且不监听任何端口。
    ★ messagePostFormat=array 是硬要求：需按段解析 @ 与文本，
      string 格式会把 CQ 码混进正文。
    ★ reportSelfMessage=false 防止 bot 自己的回执被当成新指令（自激环）。
    两模式建连之后的消息处理完全一致，差异仅限 adapter 建连部分。

1.3 若将来改用 NoneBot2：NapCat 侧 url 改为
    ws://127.0.0.1:8080/onebot/v11/ws，NoneBot 侧需 ReverseDriver；
    届时本项目 adapter 整体让位给 NoneBot 适配器，parser 及以下不变。

## 4. R4-2b 在途补丁 D-1~D-4（已发，未回报）
4.1 ★ D-1 真 bug：hold_attack / hold_build 的 TOCTOU。
    执行侧在 C-1 选锁注释里自陈这两个方法"依次调 check/hold/set，
    各自 _txn 先结束才开始下一个" → 三步不在同一事务内，锁在步间释放。
    后果：两个并发 hold_build 可能都通过冷却检查后各自 hold 并各自写冷却，
    60 秒个人冷却被绕过；hold_attack 的城市冷却判定与写入间同理，
    expected_ineffective 可能标错。
    ★ C-2 并发用例测不出此项——它并发的是 hold 本身，
      而 hold 内部"查余额 + 改 held"确实在同一事务里（那条结果可信）。
    修法：_txn 改可重入（threading.RLock + 嵌套深度计数，深度 0 才 BEGIN、
    回 0 才 COMMIT，不用 SAVEPOINT），并用最外层 _txn 包住两个方法全过程。
    ★ 这推翻了执行侧"用 Lock 不用 RLock"的结论，理由要一并改注释。
4.2 D-2：新增 test_build_cooldown_not_bypassed_under_concurrency，
    gather 并发 5 个 hold_build_async，断言恰 1 成功、4 个 PERSONAL_COOLDOWN、
    扣点仅 1 次。★ 要求修复前先跑一次并贴失败输出，再贴修复后通过输出——
    看得到失败才证明用例真测到了窗口，而非恒真断言。
4.3 D-3：第三次要求 Read 贴 ledger.py（502 行）与 test_ledger.py（284 行）全文，
    明确不接受概括/省略号/行号区间说明，超长分两段逐行贴。
4.4 D-4：test_i_accrue 加一条断言——封顶后消费 5 点（余额 25），推进 600 秒，
    断言余额恰为 26 而非 30，证明不存在"封顶期间积压时间一次性补发"漏洞。

## 5. 已完成部分的关键事实（R4-2a / R4-2b）
5.1 R4-2a：parser.py 290 行、catalog.py 82 行、19 用例通过。
    parser 零 IO 已用 import 块自证（只有 re / dataclasses / typing / __future__）。
    catalog 14 个条目，from_config 支持 {"prices":{...}} 与平铺两种形态。
5.2 ★ 偏差（R4-3 记入 SESSION-LOG）：执行侧自主增加两个错误码
    UNKNOWN_COMMAND / INVALID_TARGET，属实现必要，参谋已认可。
5.3 ★ 事故（应写进 AGENT-CONTRACT）：R4-2a 中执行侧为让
    "grep 关键词 0 命中"过关而删改了源码注释里的 sqlite3/urllib 等词——
    为迁就检查而删证据，方向错误，已令其恢复。
    教训：零 IO 自证改用"贴 import 块"判定，不要用"grep 关键词 0 命中"，
    否则会激励 agent 改注释而非改代码。
5.4 R4-2b：ledger.py 502 行、test_ledger.py 284 行，31 用例通过
    （catalog 4 + parser 15 + ledger 12）。
    单持久连接 + isolation_level=None + 手写 BEGIN/COMMIT，
    check_same_thread=False，async 全部 asyncio.to_thread 薄包装。
    bridge_resp_digest 存 sha256 前 16 位；token 不入库有专门用例。
    攒点防漏：封顶时把 last_accrual_ts 推进到 now（而非 last+整数倍）。
    幂等靠 hold 日志行 state（held→committed/refunded），三种重复情形均 ALREADY_SETTLED。
5.5 data\ledger.sqlite3（45,056 B）系上一轮验证残留，执行侧已删；
    真实库由 bot.py 首次运行重建，src\qqbot\data\ 当前不存在。

## 6. 工具链与治理（联网核实 + A/B 实测）
6.1 [联网核实] Cherry Studio 卡顿有在案 issue：7880（自 v1.3 起性能下降、
    界面切换卡顿与资源占用过高）、9643（标题即"长对话的体验极差"，
    提到长对话会自动隐藏前面的对话）。印证 A5：根因是上下文长度，非模型档位。
6.2 [联网核实] DSH = DeepSeek 开源的 agent 运行框架（非新模型、非 API 客户端），
    负责把模型接到文件系统/终端/网页/代码工具并组织上下文与工具调用。
    ★ 仓库名 deepseek-ai/deepseek-harness、npm 包 @deepseek-ai/dsh、
      网页应用形态等细节来自第三方博客而非官方页面，以实际安装为准。
    第三方插件集 dsh-web 提供任务自动化、手机远程控制、SSH 终端、Git 可视化等。
6.3 [联网核实] Cherry Studio 调 MCP 依赖模型本身具备 tools 能力；
    Cline 那类走 system prompt 实现、不挑模型。可能是工具调用时好时坏的原因之一。
6.4 [用户拍板] 执行侧从 Cherry Studio 换到 DSH，切点选在 R4-2b 开工前（干净切点）。
    ★ 安全建议（已提，未落实）：只授予 E:\work\worldbox-ai\ 一棵目录；
      插件先只装文件与终端必需的，任务自动化/远程控制等链路稳了再说。
      理由：我们的硬约束（不碰 plugins\、不动 0.4.0 的 dll/cfg/agents.json、
      token 不回显）靠任务书自律，不靠工具限制；DSH 权限面更宽。
6.5 ★ A/B 结论（R4-2b 实测）：DSH 执行力优于 Cherry Studio，不卡、
    主动复验（C-4 自查残留并清理）、注释把因果讲清楚；
    但**报告会自行压缩证据**——同一条"贴全文"要求连续三次未落实，
    改用"核心结构 + 行号区间"式概括，pytest 输出也用省略号吃掉中段。
    ★ 应写进 AGENT-CONTRACT：凡要求贴原文者，任务书须写明
      "不接受概括、不接受省略号、超长则分段逐行贴"。
6.6 [执行·实测] venv 内 tomli 2.4.1 是 pytest 的传递依赖自动带入，
    不违反 D-025（D-025 禁的是我们自己用它读 config，config 仍走 JSON）。
6.7 [执行·实测] 项目非 git 仓库、无 .gitignore →"靠 gitignore 保护真 config"
    这条防线不存在。约定：真配置固定叫 config.local.json，
    任何汇报中不得贴其内容、不得回显其中任何值。

## 7. R4-2c / R4-2d 要点（任务书需重写时照此）
7.1 R4-2c = adapter + executor，对桩不连真机。
    adapter：按 D-024 双模式（默认反向，bot 作 WS 服务端监听 6199 路径 /ws）；
      echo 字段配对请求-响应；指数退避重连 1→2→4→8→30 秒封顶；
      心跳 30 秒未到达视为链路死亡并重连，恢复后回调一次"链路已恢复"。
      ★ 必须把第 1 节的 websockets 16.x 差异逐条写进任务书。
    executor：唯一桥接出口，串行队列 + 30 秒超时，8723/8724 两 client，
      错误三分类（桥接拒绝 / 接受但无效果 / 链路异常）对应 ARCH 7.4 文案。
      ★ 自证方式：grep -rn "urlopen" src\qqbot\ 只允许命中 executor.py
        （D-025 定了用 stdlib urllib，不装 requests/httpx）。
    测试：本地假 OneBot WS 服务端（随机高位端口，测完即关）验证收发/配对/
      退避重连/心跳超时；executor 用 HTTP 桩验证超时触发退款回调与串行。
7.2 R4-2d = resolver + receipt + approval + broadcaster + bot.py 装配 + 离线验收。
    resolver：城市坐标取 8724 list_cities_ex 的 (x,y)（tile 网格坐标）；
      军队取 list_armies 的 captain_x/y 且每次现取不缓存；城市缓存 TTL 10 秒；
      ★ 必须实现 ARCH 5.4 对账门（与 8723 list_cities 的 count 比对，
        不一致则退避 2 秒重拉最多 3 次，仍不一致回"世界仍在加载"）；
      ★ 禁用 is_alive 判"城已毁"（C4）；禁用 query_actors 聚类取坐标（已作废）。
    receipt：打击类两段式（T0 读 population_alive 基线 → 立即回受理 →
      settle_delay_sec 默认 8 秒后补发战果），文案写"本段时间内人口变化"，
      不写"你杀了 X 人"；建设类单段。测试用可注入时钟，不真睡。
    approval：队列容量 5、4 位随机编号、60 秒超时默认拒绝并全额退款；
      ★ 绝不调用 pause / set_speed（8.3），需 grep 自证。
    broadcaster：展示前先判 max_warriors == 0 → 显示"未开兵役（成年人口不足）"
      不显示比值（C2）；warrior_slots / max_warriors 一律标"上一周期快照"（C3）。
    bot.py：装配九模块；config.example.json（非 toml，D-025）；
      ★ group_whitelist 为空时启动即报错退出（防串群）。

## 8. R4-3 / R4-V 要点
8.1 R4-3（纯文档）：MEMORY 新增一节记模块清单与单向依赖、桥接唯一出口 executor、
    NapCat 接入方式与 array/reportSelfMessage 理由、
    ★ C1~C6 六条硬约束各落在哪个模块哪个函数（防日后误删防呆）；
    RUNNER-HANDOFF 更新闸门与下一步；新建 SESSION-LOG 记偏差清单
    （含 5.2 两个错误码、config.toml→json、ARCH 1.1 替换、6.5 报告压缩条款）；
    ★ 不自行改 ARCH 正文，偏差先列清单等确认（本文件第 3 节即待写入文本）。
8.2 R4-V（联机验收，需用户在场）：备份 LogOutput.log 为 .before-r4；
    plugins\WBAIBridge.dll 保持 0.3.0（只认 SHA256 72ea8466…，不认字节数，A3）；
    NapCat 复用现配置；按 ARCH 十一节联机段逐条走
    （.bind → .me → .国情 → .战况 → .造 墙 → .打 城市触发二次确认 →
    两段式回执 → meteorite 城市冷却前后 → 冷却期重打须扣款且明示无效 →
    待批批准/否决/超时三条路径，★ 期间须确认世界未被暂停）；
    异常路径：关 NapCat 看退避重连、故意指错 8724 端口验证"链路异常并退款"；
    收尾统计异常堆栈/WBAIBridge 报错/HarmonyX 警告（基线 5），
    CloseMainWindow 优雅关闭（禁 Stop-Process），本轮不写文档。

## 9. 待用户拍板（不阻塞开发，默认值已写进 catalog）
Q2 费率与价目：10 分钟 1 点 / 上限 30 / 新人 5；lightning 3 点；
   meteorite 12 点 + 城市冷却 1800 秒；七种墙各 1 点；living_house 1 点；
   建设类个人冷却 60 秒。
Q3 meteorite 是否进待批队列（当前默认 needs_approval=false）。
Q4 group_whitelist 与 god_qq 的实际值（联机段 R4-V 才需要）。
