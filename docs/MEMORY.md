# MEMORY — 长期记忆（Claude 重置后的恢复依据）

> 累加式维护：每轮收工必更新。每条注明日期。
> 始建于：2026-09-14（任务书 2026-09-14-A）
> 写作要求：写「为什么」不只写「做了什么」；每条技术结论标注
> "实测验证" / "文档或社区说法"；被推翻的旧结论不删除，只标注推翻日期与原因。

---

## 1. 环境与路径（2026-09-14）

| 项 | 值 |
|---|---|
| 游戏 | WorldBox 0.51.2（学习版）E:\game\worldbox |
| 存档 | <WORLDBOX_SAVE_DIR> |
| 项目根 | E:\work\worldbox-ai\（docs/downloads/extracted/backup/snapshots/logs/screenshots/src） |
| 多项目根 | E:\work（勿在根下裸放本项目文件；PROJECT_STATE.md 下划线版属 MC 项目） |
| 项目交接文档 | docs\WORLDBOX-PROJECT-STATE.md（唯一权威状态文档） |
| Agent 侧记忆 | AGENT-CONTRACT.md / MEMORY.md / DECISIONS.md / SESSION-LOG\ |

## 2. 已装组件（版本 + 哈希，2026-09-14）

### NML（NeoModLoader）1.2.0.1 + 0.51.2 非官方兼容补丁 v1.1 — 实测验证可用
- 包：neomodloader-worldbox-0512-_patch_11.zip，8,863,499 B，MD5 79e84b29e9e5e42dcdb4da203be7e9eb
- 安装位置：worldbox_Data\StreamingAssets\mods\ 下两个 dll：
  - NeoModLoader.dll — 22,558,208 B，SHA256 92F1D3368B119028647B881EB855C47F006526259353387F2976FAB9FEA39C75
  - NMLManualLoader.dll — 5,120 B，SHA256 29354ECF0EAAA73F6CBC09A5D924AAF6E66DBF079C1569BEC39CE4A7667E96C2
- 为什么两个都要：官方 NML 在 0.51.2 上启动崩溃（WrappedPowersTab/TabMain.InitTab 空引用）；
  补丁版修复之，NMLManualLoader 负责在 0.51.2 上手動初始化 NeoModLoader（1.2.0.1 内置加载器自初始化失败）

### EmpireCraft 0.4.2（帝国盒子，长明火_CHM）— 实测验证可用
- 位置：E:\game\worldbox\Mods\EmpireCraft\（注意：游戏根目录 Mods，与 StreamingAssets\mods 职责不同，极易混淆）
- 纯源码模组 1531 个文件，0 dll；NML 用 Roslyn 现场编译，产物在
  StreamingAssets\mods\NML\CompiledMods\（长明火_CHM_EMPIRECRAFT_帝国盒子_.dll 1,124,352 B）
- mod.json 无 targetGameVersion 与 dependencies 字段（实测）

### BepInEx 5.4.23.5 + WorldBoxBridge v0.4.0 — 2026-09-14 安装并验证通过 ✅
- BepInEx 5.4.23.5（win x64）：zip 639,118 B，SHA256 82f9878551030f54657792c0740d9d51a09500eeae1fba21106b0c441e6732c4（GitHub API digest 与实测一致）
  - 安装位置：E:\game\worldbox\ 下 winhttp.dll + doorstop_config.ini + .doorstop_version(4.5.0) + BepInEx\（core 18 文件 = 12 dll + 6 xml）+ changelog.txt；顶层 4 + core 18 = 22 文件全量复制，逐文件 SHA256 比对通过
- WorldBoxBridge v0.4.0：zip 80,225 B，SHA256 98f0c5467a91ab3cbb62abc5c97274e0a54d68a47f4faefaf50306b988db68da（API digest 与 .sha256 文件与实测三方一致）
  - 安装位置：E:\game\worldbox\BepInEx\plugins\WorldBoxBridge.dll（187,904 B，SHA256 C529200653D42C6562BB2A30D2EA88532D122A0D7C5C56281AB34AB5BBFEB7BD）
- ★ 0.4.0 实际提供 28 个 HTTP 命令（较新版 29 个，0.4.0 少 1 个——这是选稳定版的已知代价，非故障）
- 桥接监听 http://127.0.0.1:8723，legacy 单 token 模式；/health 实测 ok=true，返回 assembly_csharp_sha256 与基线一致
- ★ token 存放位置：E:\game\worldbox\BepInEx\config\WorldBoxBridge.cfg（值不写入任何文档；cfg 字段：enabled=true、host=127.0.0.1、port=8723、token len=48；另有 [Game] suppress_startup_window=true）
- ★ 废弃版本留档：downloads\mcp-bridge\deprecated\（BepInEx 5.4.23.2、WorldBoxBridge 0.6.0 及各自元数据）——不要再用
- ★ BepInEx 同时看两个日志：E:\game\worldbox\BepInEx\LogOutput.log + 存档目录 Player.log
- 回滚方式（已商定）：删除游戏根的 winhttp.dll、doorstop_config.ini、.doorstop_version、BepInEx\ 即可（安装前确认过这些文件原先不存在，无覆盖风险）
- ★ 新快照：snapshots\snapshot-working-nml-empirecraft-bepinex\（1,595 文件 53.1 MB，含 BepInEx 与 doorstop、两份日志基线、MANIFEST）

## 3. 关键基线指纹（2026-09-14，实测）

- Assembly-CSharp.dll：SHA256 `51D275F0168BE2F6CA26341AB292406714E694E0270EAFCB25B999D5DF6DD69F`
  大小 4,535,296 B，时间戳 2025-09-13（判定为官方原始构建）
  - 实测（BepInEx 安装前复核，2026-09-14 14:08）：仍与该值逐字一致
- 回滚快照：snapshots\snapshot-working-nml-empirecraft\（1,578 文件 59.7 MB，含 MANIFEST.txt + 100 年测试档）
- 备份：backup\（Managed 293 文件 42.3 MB + mods + saves 303 文件，共 597 文件 351.7 MB）
- 存档目录 Player.log 系列改名基线：Player.log.before-nml / .before-empirecraft（存于游戏存档目录与 logs\）

## 4. 已知问题及影响面（2026-09-14，实测验证）

1. NML 三个事件监听器构造失败（0.51.2 类结构变化）：WarStartListener / PlotStartListener /
   ActorTryToAttackListener → 任何依赖 NML 事件系统监听"战争/攻击"的模组该功能会静默失效。
   EmpireCraft 不受影响（自实现战争前线逻辑）。
   推测（未验证）：这是社区报告 RulerBox/KingBox 在新版上"法律改不了、贸易不工作"的根因。
2. TooltipLibrary.showTrait 的 Harmony 补丁 IL 编译失败 → 仅影响悬停提示。
3. EmpireCraft 三个 locale CSV 加载失败（Culture_Corporate：CorporateEnglishLetters/GreekLetters/Number）→ 仅影响部分派系随机命名。
4. EmpireCraft 社区反馈潜在风险（未复现，长期观察中）：后期卡顿/存档失败；有用户怀疑族谱功能是卡顿主因，
   建议在模组设置里找开关。作者 README 自承 balancing/performance 仍在进行中。
   已观察证据：100+ 年无卡顿、存读档正常（用户体感）。
5. ★ 未知风险：NML（NCMS 注入）与 BepInEx（doorstop 注入）两套机制的共存 —— 官方无任何说明，需实测。

## 5. 版本兼容性要点（2026-09-14）

- 官方 NML 在 0.51.2 必崩 → 必须补丁版 v1.1（见第 2 节）
- NCMS 已停维护（仅支持到 0.21.1）；0.22 时代"开盘辅助 mod"在 0.51.2 不可用（Claude 侧结论）
- worldbox-mcp 桥接按 GameBanana/README 说法针对 0.51.2 开发（其兼容性表自测 0.51.2 ✅）
- BepInEx 用 5.x（Mono x64），默认 v5.4.23.2；WorldBoxBridge 对应 Unity 2022.3.60f1 Mono
- ★ 版本未决情报（来源：任务书 2026-09-14-A 第 5 节，作者 compatibility.md，我未亲验）：
  WorldBoxBridge v0.6.0 标「CI 通过但未经真机验证」；唯一「✅ 真机验证通过」是 0.4.0；
  0.3.0–0.3.3 曾发布无法加载的 DLL。
  ★ 已作废（I 轮标注）：最后一句「是否改用 0.4.0 首装由用户决定，未指示前不下载不安装」
  已于 B 轮作废——0.4.0 已在 2026-09-14（B 轮）安装并验证通过（见第 2 节与 D-008）。
  本条目其余版本情报（0.6.0 未真机验证等）仍有效。

## 6. Cherry Studio 模型槽配置（2026-09-14）

- Agent 槽（当前执行任务书的槽）：modelscope::deepseek-ai/DeepSeek-V4-Pro-0813
  - ★ 实测：该模型不接受图像输入（Read 截图时返回 "this model does not accept image input"），
    因此历轮"截图验证"实际靠窗口句柄/进程/日志间接取证，从未直接"看"过画面
- 其余两个模型槽（主脑/干活/备用分工、供应商、代理、费用）：
  ★ Agent 侧无信息来源，全部未知 —— 需 Claude 或用户补充。
- 视觉模型测试结论：★ I 轮改写（旧「未验证」结论已过时，此处区分两件事）：
  ① **主槽**（V4 Pro，本 Agent 所在槽）确实**不接受图像输入**——历轮"截图验证"
  实为窗口句柄/进程/日志间接取证从无例外；
  ② **Flash 槽视觉能力已由用户实测可用**（AMD TokenFactory 端点，
  DeepSeek V4 Flash Vision Exp），详见 ★ 第 8、9 节。
  两者并不矛盾：收图能力按槽区分，不是按模型供应商一刀切。

## 7. 待办与闸门（2026-09-14 C 轮更新）

- ✅ 已过闸门：BepInEx 5.4.23.5 + WorldBoxBridge v0.4.0 已安装并验证通过
  （LogOutput "listening on http://127.0.0.1:8723 (commands=28)"；/health、/capabilities 200；
  NML + EmpireCraft 加载正常，两套注入不打架）。
- ✅ C 轮完成：Python 侧 MCP server 已装并冒烟通过
  （venv：E:\work\worldbox-ai\src\mcp-server，worldbox-mcp==0.4.0，CPython 3.11.15；
  29 个 MCP 工具；capabilities 返回与基线一致）。
  ★ stdio 启动环境变量实测结论：
    - 仅设 WORLDBOX_MCP_WORLDBOX_DIR=E:\game\worldbox 即可（server 自动从 cfg 读 token），
      这最适合 Cherry Studio 配置（不用暴露 token 值）
    - mcp SDK 的 stdio_client 用 env 白名单（get_default_environment），自定义变量
      必须通过 StdioServerParameters(env={...}) 显式传，os.environ 直接设可能不生效
  ★ EmpireCraft 资产可见性（关键结论，实测）：
    - list_powers 359 项中**含 13 个 EmpireCraft 帝国/爵位 power**（create_empire、
      create_title、add_title、destroy_empire_core、Empire_layer、empire_enfeoff 等）✅
    - list_actors 322 项：civ 条目 70 个全为原版 civ_*，**未见** culture-flavored
      新种族条目（easternHuman/huaxia 等 0 命中）——如实报告为"部分可见"：
      能力类资产可见，actor 种族类资产不可见
  ★ 其他实测结论：0.4.0 server 的 stdio 客户端需基础字段；server 打印日志用 structlog；
    Bearer 是正规鉴权头（manual.md 明确 server 发 Bearer；X-WB-Token 是 mod 侧兼容头）。
- ★ MCP 接入实际通路（R0-3 已查实）：Cherry Studio **从未配置** worldbox MCP
  （sqlite 的 mcp_server 表仅 @cherry/python、@cherry/browser、blockbench、minemap 4 条，
  worldbox 仅命中聊天全文索引），历轮实测走的是 `src\mcp-server\` venv 直连脚本
  （StdioServerParameters(env={"WORLDBOX_MCP_WORLDBOX_DIR": "E:\\game\\worldbox"})，
  仅设该变量 server 自动从 cfg 读 token，无需暴露 token 值）；
  配置 Cherry 运维台 MCP 仍是待办。
- 下一步：推进 QQ bot（NapCat / OneBot v11，直连 bridge HTTP /cmd，不走 MCP）。
- ⚠️ 未决观察项：WorldBoxBridge 日志中 **5 条** HarmonyX Warning（G 轮更正确认，性质判定）：
    - System.Array.Empty ×3：HarmonyX 常见无害探测
    - AssetManager.loyalty：字段在 0.51.2 上路径变化，影响面待定
    - ★ WindowMetaGeneric OnEnable：实质风险提示，补丁可能静默不生效——
      G 轮实测判定：依赖 UI 层的三处能力（get_ui_state / dismiss_window /
      suppress_startup_window）均实测生效，该警告未造成可见功能损失（见第 15 节）
- ★ 勿升级 EmpireCraft：0.4.3Beta1 更新提示来自 mod 内置自更新推送，GitHub 无对应发布物，
  渠道不明且为 Beta；维持 0.4.2 不动。
- ★ PyPI 版本事实核查（Claude 侧说法修正）：PyPI 最新为 0.6.0；
  「PyPI 已到 0.7.0」系误报（0.7.0 只有 GitHub tag，未发布到 PyPI）。
- ★ 快照核查结论（C 轮）：新快照 snapshot-working-nml-empirecraft-bepinex 不含 testsave
  （12 文件 8.9 MB：year102 手动档 + year121 自动档）；其余（BepInEx 全套/NML 两 dll/
  EmpireCraft 源码）逐字节与旧快照一致；存档本体仍在游戏存档目录未动。待用户决定是否补。
  体积 -6.6 MB 与文件数 +17 均已逐项解释清楚（-testsave 8.9MB + BepInEx 2.3MB）。

## 8. 玩法定位（2026-09-14 用户拍板，重大修正）
- ★ 终局玩法 = **投屏 + 群友共同参与的 RTS（即时战略）**，不是回合制。
- 旧方案「回合制推进 + 截图发战报」**已于 2026-09-14 作废**（文档曾长期记录过时方案
  导致 Claude 反复设计错方向——这是文档事故，已修复；旧条目保留划掉+作废标注）。
- 观察通道 = 投屏画面；GM **不推送战报**（该需求不存在）。
- 节奏 = 世界持续运行；不做回合闸门。并发/限流设计需按 RTS 模式重做。
- Flash 视觉模型用途 = ①让 agent 能看见画面更好工作；②成本便宜可高频调用。
  不是用于战报。
- Flash 视觉模型（实测可用）：AMD TokenFactory 端点，DeepSeek V4 Flash Vision Exp，
  能正确读出中文 UI 标签与面板内容；限制：上下文 20 万 tokens，高频截图会快速吃掉上下文。

## 9. 模型与平台事实（2026-09-14 更正）
- Cherry Studio = 单个智能体、**三层模型槽**（主 / Plan / Small），当前为
  V4 Pro (0813) / V4 Pro (0813) / V4 Flash。★ 不是三个独立 agent（旧记法已更正）。
- 本 Agent 跑在主槽 V4 Pro（ModelScope 来源）；**不能接受图像输入**（Read 截图返回
  "this model does not accept image input"）——此前所有"截图验证"实为窗口/进程/日志间接取证。
- Cherry 是否把图像自动路由到 Small 槽：**未知、待实测**，不要假设可以。
- 视觉模型走 AMD TokenFactory（OpenAI 兼容端点）；DeepSeek 官方 API 已退役
  v4-flash-vision-exp 由 deepseek-flash 承接，但第三方平台挂固定权重快照，
  官方退役不影响第三方。★ 旧推断"AMD 端点可能静默降级纯文本"已被用户实测推翻——勿再重复。
- token / bearer 一律不入文档不入汇报。

## 10. 桥接架构关键约束（2026-09-14 E 轮）
- 三进程结构：AI client ↔ MCP server ↔ bridge in worldbox.exe（bridge 注入游戏进程）
- ★ 所有游戏操作经 ConcurrentQueue 派发到 Unity 主线程逐帧串行执行；dispatcher 有 30 秒硬超时
- ★ E 轮串行耗时实测（31/0/16/0/16 ms）判为**无效测量**（G 轮更正）：全部为 15.6 ms 整数倍
  = Windows 默认时钟节拍（低精度计时器）；0 ms 的 HTTP 往返不可能真实存在；
  更根本的是顺序调用无法验证串行性（队列排队只有并发请求才暴露）。
  有效结论：低负载下单命令延迟很低（量级为数十毫秒），距 30 秒超时余量极大；
  但「所有操作串行经 Unity 主线程」这一性质**仅有上游架构文档依据，尚未实测验证**
  （保留未验证，不做并发压力测试——风险不值得）
- 监听实现用 TcpListener 而非 HttpListener（后者在 Unity 2022.3 Mono 下静默绑定失败）
- ★ 0.51.2 上 BepInEx 创建的 MonoBehaviour 会在 Awake 后不久被销毁，作者改用 PlayerLoop 注入
  （将来自写桥接命令会撞上这个坑）
- 反射查找失败只禁用受影响的单个命令，其余照常工作（优雅降级）
- 鉴权：Authorization: Bearer 为正规头；X-WB-Token 为兼容头（bridge 先试 Bearer 再回落）
- ★ 任务一结论：mod 侧注册 28 个 command；29 个 MCP 工具 = 28 command + 1 个 capabilities
  （capabilities 是 server 侧的本地工具，不映射到 mod 的 command，属正常设计非缺失故障）

## 11. 多 agent session 层（2026-09-14 E 轮，核对上游 v0.4.0 文档 + 本地 README）
- turn_based 默认 false，可关闭，即时战略可行
- 四种 scenario 预设：pvp / coop / hierarchical / sandbox，切换只需改 JSON
- ★ 0.4.0 的 faction_player 可以调用 invoke_power（属 ActionFaction 权限，与 spawn 同组）；
  只有 paint_tile 等全图生效的属 ActionGlobal
- AdvanceTime 权限（pause/resume/set_speed）给了 faction_player（作者本意是 PvP agent 可自行快进暂停）
- ControlWorld（generate_world/save_world/load_world）god-only 且写死
- partial_intel 为 true 且无 ReadAll 权限时：list_cities 与 query_actors 按所属王国过滤；
  list_kingdoms 故意不过滤；★ screenshot 直接返回 PERMISSION_DENIED（防靠截图绕过迷雾）
- 每 agent 收件箱 200 条上限，超出丢最旧；序列号全局，单个 since_seq 游标即可
- ★ 文档中未见任何速率限制/配额/冷却的原生支持字段 → 需在 Python 侧自行实现
- agents.json 一旦存在即退出 legacy 模式；legacy 为 id="legacy"、role=God、全权限、
  token 取自 WorldBoxBridge.cfg
- 新增错误码：PERMISSION_DENIED(403)、FACTION_SCOPE_VIOLATION(403)、TURN_NOT_YOURS(409)

## 12. 玩法规模（2026-09-14 E 轮）
- ★ 实际参与者为朋友间娱乐，不超过 5 人，非公开大群
- 这大幅降低限流、配额、反滥用的设计优先级，相关设计可从简

## 13. 每轮收工检查单

- [ ] MEMORY.md 更新（本轮新事实/结论）
- [ ] DECISIONS.md 增补（有决策产生时）
- [ ] SESSION-LOG\YYYY-MM-DD-X.md 新建
- [ ] 汇报中原文贴回关键输出，区分实测/推测

## 14. 各轮运行模型与任务书字母序（2026-09-14 E 轮补记，J 轮补序，R0 轮更正）
- ★ 任务书实际收到序列：**A → B → C → D → E → G → I → J → L → M → N → O → P → Q-MEM**。
  D 轮任务书送达过（产出 D-007~D-009）但**无 SESSION-LOG**；
  **F / H / K 三个字母从未送达** Agent（F 被 E 替代/覆盖，H 在传输中丢失，K 未收到）；
  **Q-MEM 只有收工报告、无 SESSION-LOG**（§13 检查单第 3 项漏项）。
  今后引用轮次以此序列为准，不要去找 F/H/K 轮的记录。
- B 轮：V4 Pro 0813（主模型）
- C 轮：V4 Pro 0813（主模型）
- E 轮起：DeepSeek V4 Flash（纯文本，1M 上下文）——主模型额度耗尽后切换，
  主模型与 Plan 模型均已切换；历史报告可靠性需按当时运行模型评估
- ★ 日期错位事实（R0 轮补记）：SESSION-LOG 文件名与文档头的 2026-09-14 系任务书字母沿用，
  **真实执行日期跨 09-14 至 09-16**（证据：baselines README 记 2026-09-15；
  Q-MEM 收工时间 2026-09-16 20:35）。今后 SESSION-LOG 以真实日期命名，字母写在正文。
- ★ Cherry 框架侧 FACT.md（Cherry Studio 数据目录下的跨项目记忆）存在 worldbox 相关条目，
  其中部分为 G 轮结论的旧版本（如「UI 三能力全部✅生效」未含 I/J 轮正路径更正）。
  不与框架文件同步维护；**与本项目结论冲突时以 docs\ 为准**（2026-09-14 J 轮约定）。

## 15. UI 层三能力实测（2026-09-14 G 轮 ● J 轮已更正，正路径见第 16 节）
- 背景：5 条 HarmonyX Warning 中 WindowMetaGeneric OnEnable 属实质风险（补丁可能静默不生效）。
- ★ G 轮结论证据强度更正（J 轮）：
  1. suppress_startup_window = true：✅ **生效**（LogOutput 原文 suppressed 行 + 无窗口互证）。
     此条证据链完整，为真"已生效"，无需更正。
  2. get_ui_state：G 轮「✅ 生效」→ 更正为 **仅验证负路径**（四字段全 false，与桩函数
     观测不可区分，不能证明生效）；正路径实测见第 16 节（I/J 轮）。
  3. screenshot：✅ **生效（视觉通路可用）**，此条为真"已生效"（返回了真实帧数据）。
     ★ 字节差解释更正：G 轮调用了两次 screenshot，元数据 bytes=161426 与落盘 161,507 字节
     是**不同两帧**的 JPEG 编码结果；原「base64 解码后不完全相同属正常」的解释**错误已作废**。
  4. dismiss_window：G 轮「✅ 生效（语义正确）」→ 更正为 **仅验证负路径**
     （无窗口时返回 dismissed:false 只证明调用未报错）；正路径实测见第 16 节。
- ★ 旧结论「WindowMetaGeneric OnEnable 警告未造成这三处可见功能损失」**已作废**
  （G 轮仅测负路径，证据不足）；以第 16 节 J 轮 Kingdom 面板正路径实测为准。

## 16. UI 状态检测正路径实测（2026-09-14 I/J 轮）
- ★ 方法：I 轮把 suppress_startup_window 改为 false 制造真实 welcome 窗口；J 轮观察到
  Kingdom 面板被用户打开（基线即 window_active=true, current_window="kingdom"）。
- **welcome 窗口（I 轮，正路径）**：
  - get_ui_state：#1 → {"window_active": true, "current_window": "welcome",
    "config_paused": false, "effective_paused": true, "world_loading": false}
  - dismiss_window → {"dismissed": true, "window": "welcome"}
  - get_ui_state #2 → 全部回 false
  - ✅ 检测与关闭机制均可用
- **Kingdom 面板（J 轮，正路径，最关键的证据）**：
  - get_ui_state → {"window_active": true, "current_window": "kingdom",
    "config_paused": false, "effective_paused": true, "world_loading": false}
  - dismiss_window → {"dismissed": true, "window": "kingdom"}
  - 之后 get_ui_state → window_active=false, effective_paused=false
  - ✅ 判定一（检测）：Kingdom 面板打开时 window_active **翻 true** → WindowMetaGeneric
    警告**未造成检测缺口**，G 轮结论意外正确；
  - ✅ 判定二（暂停）：Kingdom 面板打开时 effective_paused=**true** →
    世界停摆（与任务书推测"false，边看边跑"相反——以实测为准）。
  - ★ RTS 意义：打开国家面板会冻结全场。若朋友玩家 AI 或脚本打开面板不关，
    世界会永久停摆。RTS 设计需处理（dismiss 兜底/限制面板操作/定时 dismiss_window）。
- ★ 关键证据记法：current_window 字段**仅在窗口打开时出现**，字段集合随状态变化
  → 可排除桩函数假设（桩函数不会动态增删字段）。
- ★ 勘误：'welcome 窗口暂停世界'**不可推广**为'任何窗口都暂停世界'——welcome 是启动阻塞
  对话框；Kingdom 面板是游戏内窗口，两者机制不同。但恰好两者实测 effective_paused 都为 true。
- screenshot 参数（/capabilities 实测 schema）：max_dimension(integer,min 0,default 1280,
  最长边像素、0=全分辨率不缩放)；format(jpg|png,default jpg)；quality(1-100,default 80,
  仅 jpg 生效)。J 轮实测字节量：jpg q80=199,928 B / png=1,151,851 B / jpg q95=425,889 B
  （均为 1280×705）。推荐值（J 轮）：max_dimension 默认 1280（不超 Flash 视觉约 1300 阈值
  不被二次缩放）；format/quality 取舍由视觉模型实测决定——png 无损但体积 ~5.8 倍，
  同尺寸下 token 按像素算，png 是"免费"清晰度提升的候选。
- ★ cfg 编辑编码陷阱（I 轮踩坑）：PS 5.1 的 Get-Content -Raw 按 ANSI 解码，
  非 ASCII 字符（如 em-dash）在读取阶段即丢失；改 cfg 必须用
  [System.IO.File]::ReadAllText 读 + WriteAllText 配 UTF8Encoding($false) 写，
  并先在副本上做"读-写-比哈希"往返验证再动真文件。
  ★ cfg 备份含 token，不得留在 logs\ 等会被整目录列举的位置。I 轮备份已删除（J 轮确认）。
  ★ 正确回滚方式（J 轮已验证）：Copy-Item 整字节拷回 + SHA256 比对，勿编辑。
## 17. 资产与性能基线（2026-09-14 L 轮，部分完成）
- ★ 资产基线已落盘：E:\work\worldbox-ai\baselines\assets-vanilla-empirecraft\
  （powers.txt 359 条 4,129 B / actors.txt 322 条 5,124 B / tiles.txt 20 条 211 B /
  speeds.txt 10 条 39 B / README.md 789 B，另含采集口径说明）。
  采集方式：MCP 四工具（list_powers/actors/tiles/speeds）现场取值、排序去重、每行一个 id；
  采集时点为 NML+EmpireCraft 0.4.2 已装、无其他 mod；359/322 与 MEMORY 既有记录完全一致。
- ★ 性能基线：**未完成**。load_world 加载 testworlds\year121-autosave-1789319500 失败（两次）：
  - path=存档文件夹 → "[GAME_CRASH] Bad state (unknown compression method (0x7B))"
  - path=map.wbax 文件 → 同上错误
  - 失败时游戏进程未崩溃（world_state 仍可正常返回，tick 正常推进）。
  - ★ M 轮已查明（反编译实证，替换旧判读）：
    SaveManager.loadMapFromBytes 调用 Zip.Decompress（Assets.SimpleZip），期望 SimpleZip
    压缩字节流；而本机 map.wbax 是未压缩 JSON（首字节 0x7B = '{'），map.wbox 是 Ionic.Zlib
    ZlibStream 压缩——两者都不是其输入格式，故必然报 unknown compression method。
  - ★ load_world 已定性为已知缺陷、低优先级、暂不修复（M 轮修正 3）：
    理由：该命令不在玩法路径内（generate_world/load_world 永不暴露给玩家，
    RTS 为世界持续运行、无中途读档），唯一用途是搭测试场景，而该场景由用户
    在游戏内手动读档即可完成（GUI 操作归用户，见分工约定）。
    实现细节与存档格式结论见 docs\FEASIBILITY-SELFHOSTED-COMMANDS.md 段 2。
- ★ invoke_power 验证：**N 轮已完成（2026-09-14）**，替换本条旧"未完成"：
  - 世界：用户手动读档（seed=2, 8 城市/3 王国, pop 1379）；靶点：query_actors
    kingdom_id=8 取单位坐标 (208,97)（list_cities 不返回坐标）
  - 结果（6/6 accepted=true，0 被拒）：
    A 类可进商店（accepted+人口变化）：meteorite(-1) / lightning(-178) /
      earthquake(-4) / tornado(-3) / atomic_bomb(-2)
    B 类调用被接受但 3 秒无可观测效果：napalm_bomb(0)（持续燃烧型，窗口内无死亡）
    C 类被拒：无
  - via 分布：meteorite/lightning/earthquake/tornado 走 click_action；
    napalm_bomb/atomic_bomb 走 click_power_action
  - ★ 注意：同一坐标连续打击后效果递减（lightning -178 是第一次大杀，
    后几个 power 在已清空的区域只能 -2~-4）——定价时须考虑场景人口密度
  - ★ 设计事实实证：0.4.0 invoke_power 无 radius/pulses 参数（L 轮 schema 层确认，
    本轮实证补充），打击强度只能靠换 power id 分级
- ★ O 轮补充（query_actors 字段边界）：query_actors 返回仅 name / asset_id / kingdom_id /
  x / y（无职业/城市/血量字段）。按 actor 统计兵力就绪度不可行，需自写 list_armies 命令
  （M 轮已确认 Army.countMelee / countRange 为 public）。
- ★ 接口边界（L 轮实测确认，可复用）：
  - 0.4.0 invoke_power 无 radius/pulses 参数（radius/pulses 是 0.6.0 新增）；
    打击强度只能靠换 power id 分级（来自 /capabilities 实际 schema）。
  - get_world_state 返回 width/height/seed/tick/paused/population_alive/
    population_lifetime/kingdoms_alive/kingdoms_ever_created/cities_alive/
    cities_ever_created——无黄金/食物等资源字段。
  - 0.4.0 无移动部队的命令。
  - ★ paint_tile 接受 **tile id**（来自 list_tiles，如 mountains），**不是 power id**
    （传 tile_mountains 会返回 UNKNOWN_ASSET）——两套 id 命名空间不通用，
    调用前须以 list_tiles / list_powers 名单为准（2026-09-14 P 轮补记）。
- ★ 待办（下一轮）：load_world 与本机存档格式兼容性专项；性能基线（tick/秒）与
  invoke_power 打击验证继续。
## 18. 自写桥接命令可行性（2026-09-14 M 轮，反编译实证）
- ★ 详细报告：docs\FEASIBILITY-SELFHOSTED-COMMANDS.md（四段，13,685 B，230 行）
- 工具链：ilspycmd 9.1.0.7988（NuGet 官方 icsharpcode，net8.0，4,183,395 B，
  SHA256 2B5058F5...443AF7C）；本机已装 dotnet SDK 9.0.101 直接运行；
  Assembly-CSharp.dll 反编译 2440 文件 14 MB（decompile\src\），
  WorldBoxBridge.dll 反编译 70 文件 334 KB（decompile\bridge-src\）。
- 资源系统：**可读可写**。Building.resources（public，CityResources）→ get/change(负=扣)/set 全公开；
  City 有 getTotalResourceSlots() 聚合；Kingdom 无资源字段。现有 28 命令无资源读写，需自写。
- 军队系统：**无目标字段**。Army/ArmyData 均无 target/goal/order/behaviour 类字段或命令方法
  （仅 setCaptain）；行军靠统帅 Actor.setTask(string) 驱动，AI 自己推线（合 D-005）。
- 战争控制：**可强制宣战/停战**。WarManager.endWar(War, WarWinner) public；
  DiplomacyManager.startWar(...) internal 但 GameRefs 反射+NonPublic 可触达；newWar public。
- 自写命令模板：ICommand + GameRefs 反射缓存 + MainThreadDispatcher + Permission 门 +
  BridgeRejectionException 错误协议（桥接 0.4.0 源码全可照抄）。
- ★ 结论三档：资源消耗=可行；调兵=部分可行（AI 推线）；战争控制=可行。
  是否走自写路线由用户拍板（本轮未在 DECISIONS 记录决策，任务书修正 1 要求）。
## 19. 防御工事与军队目标机制（2026-09-14 O 轮，实测+反编译）
- ★ wall_iron 可通过 invoke_power 放置（O 轮实测）：
  invoke {"power_id":"wall_iron","x":170,"y":135,"accepted":true,"via":"click_action"}
  前后 get_tile top_id：jungle_high → wall_iron
- angle_tower / flame_tower / monolith：invoke 均返回 accepted=true（via=click_power_action）
  但 get_tile 前后无明显差异 → 调用被接受但放置不可确认
- living_house：accepted=true，actor_count 5 增加 3 个新城民，可确认生效
- ★ 军队目标机制真实位置（O 轮反编译）：
  - City.target_attack_city（internal City）：城对城攻击目标（QuantumSpriteLibrary 绘制箭头
    读它）；赋值在 CityBehCheckAttackZone.execute() 每周期 AI 决策
  - City.target_attack_zone（internal TileZone）：攻击区域
  - Actor.beh_tile_target（internal WorldTile）：单位行为目标（大量行为脚本改写），
    如 ai.behaviours/BehCityActorCheckAttack.cs 用 city.target_attack_zone 派生；
    另有 BehAnimalFindTile / BehBoat* 等大量非军事用途也写它
  - isOkToSendArmy()：army.countUnits() / getMaxWarriors() >= 0.7f —— 70% 阈值
- ★ 军队目标字段判定：都不是一次性持久命令，是 AI 行为周期派生值（每周期执行会重置/
  重新指派）→ 写"命令"字段不通，自写调兵命令须从行为层上手（插件任务）。
- ★ R0 轮补记（来自 Q-MEM 报告 M1-5 / M1-6 的「仅报告未落盘」结论）：
  ① spawn 可作强制征兵原语——⚠️ 标注为任务书断言（Q 主任务书 1-5 预期结论，STATE-BRIEF B3 已记录），
     本机 R0-4 盘点未找到 Q 轮 spawn 补测的实测日志（logs 无 Q 结果文件、DECISIONS 无 D-019）→ **待实测确认，非已证**。
  ② EmpireCraft ActorPatch 的 moveTo prefix 补丁**仍注册**，仅 ShouldBlockMoveToByFrontLine 守卫被注释
     ——P 轮 4B-3 反编译实证（ActorPatch.cs:149-152，注释掉的拦截分支）→ 实测确认。
     这是 D-017 行为层路线的直接证据：作者放弃 Harmony 硬拦截，改用新增行为节点。
## 20. 自写插件骨架与并存验证（2026-09-14 P 轮，实测验证）
- ★ 自写插件 wbai.bridge（新工程 `src\wbai-plugin\`，独立于 O 轮 plugin-skeleton）：
  - BepInPlugin GUID `wbai.bridge` / Name WBAIBridge / Version 0.1.0 —— 与原版 GUID
    `com.fullya99.worldbox-mcp.bridge` **不冲突**（原版摘自 bridge-src Plugin.cs:19）
  - 端口 **8724**（默认，cfg 可改）；独立配置 `BepInEx\config\WBAIBridge.cfg`（新文件）
    —— **完全不碰 WorldBoxBridge.cfg 与 WorldBoxBridge.dll**
  - 只引用 BepInEx.dll + UnityEngine.dll + UnityEngine.CoreModule.dll；**无游戏类型引用、
    无 Newtonsoft、无 0Harmony**（/health 响应是常量字符串）
  - 初始化走 PlayerLoop 注入（复刻 0.4.0 MainThreadDispatcher.Bootstrap，tick 挂 Update 阶段），
    **不用 MonoBehaviour**——规避 0.51.2 上 MonoBehaviour 被销毁
  - 编译：0 错误 1 警告（MSB3277 netstandard 2.0 vs 2.1 冲突，net462 固有摩擦，无害），
    WBAIBridge.dll = 15,360 B
- ★ 并存验证：**成功**。BepInEx 同时加载 WorldBoxBridge 0.4.0 + WBAIBridge 0.1.0
  （"2 plugins to load"），两端口各自监听：
  - 8723/health（原桥接）→ 200，assembly_csharp_sha256 与基线一致
  - 8724/health（自写）→ 200 `{"ok":true,"plugin":"wbai.bridge","version":"0.1.0"}`
  - 8724 无 token / 错误 token → **401**（鉴权真正生效，X-WB-Token 与 Bearer 双路径均可）
  - HarmonyX 警告仍为 5 条已知（Array.Empty ×3 + loyalty + WindowMetaGeneric OnEnable），
    与 before-wbai 基线无新增
- ★ EmpireCraft Roslyn 编译在并存下**仍正常**：CompiledMods 的
  `长明火_CHM_EMPIRECRAFT_帝国盒子_.dll` 本次启动重新生成（时间戳 14:04→18:08），
  字节数仍 1,124,352 B（与 MEMORY §2 一致）——三套注入（NML / WorldBoxBridge / wbai）
  首次共存未破坏最脆弱的 EmpireCraft 编译链路
- ★ 回滚方式：删除 `BepInEx\plugins\WBAIBridge.dll` 即可（日志基线已留存
  `LogOutput.log.before-wbai` + `Player.log.before-wbai`）
- ★ get_tile（0.4.0）字段边界（P 轮实测，5-7）：返回仅 x / y / tile_id / top_id / height /
  actors / actor_count，**无 city_id / zone / kingdom 字段** → 无法经桥接判断某坐标是否位于
  城市或领土内；「塔是否需要城市上下文」在 0.4.0 上不可验证
- ★ 塔类 power（angle_tower / flame_tower / monolith）在空地（O 轮）与单位密集区（P 轮）
  均为 B 档（accepted 但无可观测放置），**停止继续测试**；工事玩法以墙类为准
  （wall_iron / wall_ancient 实测 A 档、60 秒后与跨会话均持久）
## 21. 无生产环境，破坏性实验可直接进行（2026-09-14 Q 轮补记）
- 本项目为一次性娱乐用途（D-018）：无生产环境，存档有自动保存，
  破坏性实验（含写资源 CityResources.change/set、强制宣战等）可直接进行，
  出问题重开档即可，不必因「怕弄坏环境」而裹足不前。
- 已落地配套：worldbox-mcp 的 save_world / generate_world / load_world 仍不可暴露给群
  （一行字能清空世界），但**测试场景可直接在游戏内操作**。

## 22. 模型与额度（2026-09-14 Q-MEM 轮更新）
- DeepSeek（AMD Radeon Cloud 端点）已下架，魔搭额度当日耗尽。
- 主力模型：Qwen3.8-Flash-Next（1M 上下文）；备选 GLM-5.3 Flash；兜底 Qwen3.8-27B。
- MiniCPM5-2B 与 MinerU2.5-Pro 不适用（弃用）。
- 换模型后必须先跑冒烟测试（只读、不启动游戏、不改文件）再接正式任务。

## 23. list_armies 命令与 WBAIBridge 0.2.0（2026-09-16 R 轮，实测验证）
- ★ 自写插件 WBAIBridge 已实装 list_armies（0.2.0），DLL 25,600 B，SHA256
  6f8078317226ff7517b07882ac8f8ec73a09c53ce3297e3f646d6bcf00311b77（plugins\WBAIBridge.dll）。
  路由：GET /health、POST /cmd（body {"command":"list_armies"}）；鉴权沿用 Bearer + X-WB-Token 双路径。
- ★ count 口径陷阱（关键）：countUnits() = units.Count 不判 isAlive；countMelee()/countRange()
  判 isAlive（无武器也算近战）。三者口径不同，不得互相换算或相加代替。
  ok_to_send 判定优先调 **public 的 City.isOkToSendArmy()**（内部 = army.countUnits()/getMaxWarriors() >= 0.7f，
  getMaxWarriors = City.status.warrior_slots），不自己复现公式。
  ★ warrior_slots 并非「编制上限（槽位）」，而是**当期 attacker 公民岗位数**，
  每周期由 CityBehCheckCitizenTasks 重算；理论上限 = (int)(population_adults × getArmyMaxMultiplier())，
  实际值与 _citizens_left 取小、每周期步长 2；前置门 = world_law_civ_army 启用 且 population_adults > 15
  （hasEnoughFoodForArmy() 当前硬编码恒 true，City.cs:2367，不构成门槛）。
- ★ 隔离状态变更：0.2.0 起直接引用 Assembly-CSharp.dll + Newtonsoft.Json.dll（13.0.2，均 Private=false
  只读不复制），放弃「无游戏类型引用」隔离。类型加载失败只影响 list_armies（返回 500 GAME_CRASH），
  /health 与插件本体不受影响（命令外层 try/catch + ListArmiesCommand.Execute 内 try/catch）。
- ★ 真机实测（R2-4d，seed=2 世界 5 王国 8 城市 544 人口）：list_armies 返回 8 支军队，
  14 字段齐全无 null；count_units == melee+range（未见「尸体未清」导致 units > melee+range 的样本）；
  ok_to_send 全部 true 且与手算一致；3 并发 /cmd 全部 200 无死锁，期间 /health 仍 200。
- ★ WorldBox 正式版已到 0.51.4，本链路钉死 0.51.2，**禁止升级游戏**（所有插件/桥接/存档均按 0.51.2 验证）。
- ★ 墙类补测（R2-6，全部 A 档）：wall_evil / wall_green / wall_light / wall_order / wall_wild
  均 accepted + get_tile top_id 变为对应墙；ice_tower 仍 B 档（accepted 但 top_id 无变化，干净空地复测确认）。
- 字段链备忘：kingdom/city 经 getKingdom()[CanBeNull]/getCity()（判 null）；captain_x/y 经
  getCaptain().current_tile.x/y（WorldTile.x/y 是 public readonly int，BaseSimObject.current_tile public）。

## 24. R2-7~R2-9 实测结论（2026-09-16 R 轮，逐条带标签）
1. [执行·实测] 打击伤害为延迟结算：投放后瞬时采样 population/count_units 常为 0，
   需延迟约 5~10 秒复采。R2-9a 首发瞬时 delta=0、下次采样才入账 10。
   ★ 推论：逐发归因不可靠，只有多发合计可信。
2. [执行·实测] 暂停态干净口径：3 发 lightning 同一平民密集格合计 856→802 = 54；
   1 发 meteorite 打 actor_count=23 的格，791→715 = 76（杀伤大幅溢出落点格 → 大范围 AOE）。
3. [执行·实测] lightning 对军队为点杀：3 发打统帅格合计减员 1（第 2 发击杀统帅，45→44）。
   → 打击类对平民致命、对军队近乎无效。
4. [已作废] N 轮「lightning 首发 -178」不再作为定价锚点，疑为延迟结算导致的归因错误。
   同轮 earthquake/tornado/atomic_bomb 的 -2~-4 同样存疑，杀伤量级重置为未知。
5. [执行·实测] captain_x/y 字段准确（R2-9b 暂停态 3/3 命中统帅本人）。
   R2-8d「落点无人」系取坐标到落地之间世界推进的陈旧快照，非坐标系偏移。
   ★ 非暂停态下投放必须就近重取坐标，或接受打偏。
6. [执行·实测] pause 语义：冻结出生与移动，但死亡结算仍在推进
   （暂停态 population 持续下降 856→846→802）。暂停 ≠ 静止快照。
7. [执行·实测] 暂停态下桥接完全可用：/cmd、list_armies、get_tile、population_alive
   均正常返回，未触发 30 秒超时 → MainThreadPump 不受世界暂停影响。
8. [执行·实测] 0.4.0 的 28 命令含 pause / resume / set_speed / list_speeds；
   最慢速 slow_mo（multiplier 0.5）。此前未纳入设计视野。
9. [执行·实测] isOkToSendArmy() = countUnits()/getMaxWarriors() >= 0.7f，
   分母可为 0（attacker 槽位归零时），浮点除零得 +Infinity，判定恒 true。
   ★ 硬性防呆：任何消费方必须先判 max_warriors == 0，再看 ok_to_send。
10. [执行·实测] 墙类 7/7 全 A 档（R2-6 补测 wall_evil/green/light/order/wild 五种
    accepted 且 top_id 变更）；ice_tower accepted 但 top_id 无变化 = B 档，塔类判定闭合。
11. [执行·实测] WBAIBridge 0.2.0 长会话稳定：R2 全程 0 异常堆栈、0 插件报错，
    HarmonyX 警告仍 5 条无增长，CloseMainWindow 优雅退出无残留。

## 25. R3-0 list_cities_ex 可行性闸门结论（2026-09-17 R 轮，静态取证）
- ★ City 枚举入口：MapBox.cs:69 `public CityManager cities`（public 字段），
  CityManager : MetaSystemManager<City, CityData>，继承 IEnumerable（GetEnumerator）
  与 public readonly List<City> list——与 Army 同模式，全公开。
- ★ 城市中心坐标：`City.city_center` 是**世界坐标 Vector2**（City.cs:107，internal），
  **不可用于投点**；必须用 `city.getTile()`（public WorldTile，City.cs:386）的 x/y——
  **tile 网格坐标，与 invoke_power 同坐标系**。getTile() 标 [CanBeNull]，返回 null 时填 null；
  首次调用触发 recalculateCityTile 重算（City.cs:395-437，仅遍历本城 buildings+zones，毫秒级），
  **禁止传 pForceRecalc=true**。
- ★ City 无 getKingdom()（grep 0 命中）：kingdom 经 `city.data.kingdomID`（public long，
  CityData.cs:31）+ `World.world.kingdoms.get(id)`；野区 kingdomID=-1 时 get(-1) 经
  hasValue 守卫返回 null（SystemManager.cs:40），kingdom_name 填 null。
- ★ 判定：list_cities_ex **全公开实现，无反射需求**；14 字段全可拿（id/name/x/y/kingdom_id/
  kingdom_name/population/population_adults/warrior_slots/max_warriors/has_army/army_id/
  army_max_multiplier/has_enough_food_for_army/is_alive）。

## 26. R3-V 真机验证结论（2026-09-17 R 轮，逐条带标签）
1. [执行·实测] 精确坐标 vs 聚类近似偏差 0~56 格且无规律
   （id=1 偏 8.0、id=2 偏 56.4、id=4 偏 0.0）→ 聚类近似作废，
   "我城/敌城"改用 list_cities_ex 的 getTile().x/y。
2. [执行·实测] lightning 对城市近乎无效：打 202 人城市精确中心 delta=-4。
   根因：暂停态 get_tile 显示居民多在室内（actor_count=0），点杀打空格子。
   → lightning 仅对野外裸露单位有效（R2-9a 击杀统帅）。
3. [执行·实测] meteorite 是唯一洗地手段：同一精确中心 1 发 delta=-180，
   城市 population 202→20。大范围 AOE 且摧毁建筑，室内居民一并清除。
   ★ 平衡警告：单发即可清空一座中型城市，必须高价 + 长冷却。
4. [执行·实测] 城市极难消灭：小城 16 人连挨 6 发 meteorite 降至 2 人，
   is_alive 仍为 true。isRekt() 判定依据是建筑全毁（isReadyForRemoval），
   非人口归零。★ is_alive == false 本轮从未观测到，该边界仍未验证，
   不得用 is_alive 作为"城已毁"信号。
5. [执行·实测] warrior_slots / max_warriors 会滞后：id=3 城 population 202→20、
   population_adults=11，warrior_slots 仍为 112（暂停态不重算）。
   ★ 这两个字段只能当上一周期快照，任何实时判断不得依赖。
6. [执行·实测] Infinity 陷阱实机确认：population_adults ≤ 15 的四座城
   （id 13/14/15/16）warrior_slots=0、max_warriors=0，对应军队 ok_to_send 全 true。
   同时确认 R2-8e 静态推演的 population_adults > 15 门槛成立。
7. [执行·实测] list_cities_ex 可静默返回不完整列表：世界加载中首拉 count=1，
   复拉 count=6，与 8723 list_cities 对齐。代码无缺陷，但消费方必须防：
   与 8723 list_cities 的 count 对账，或世界加载后延迟首拉。
8. [执行·实测，中等强度] 野区路径已实际执行：首拉条目为
   kingdom_id=-1 / kingdom_name=null，未抛异常。保留：当时世界仍在加载，
   王国归属可能尚未落定。
9. [执行·实测] 16 城字段与 8723 list_cities / list_kingdoms / list_armies
   三方交叉对账 0 差异；坐标 16/16 指向城市区域（9 城 top=road），无海洋/空地误指。
10. [执行·实测] 0.3.0 稳定：并发 4 请求全部 200（3.9~17.9ms）无死锁，
    LogOutput 0 异常堆栈、0 插件报错，HarmonyX 仍 5 条无增长。

## 27. QQ bot v1 离线骨架（2026-09 R4 轮）
> 范围：R4 轮离线完成的八个模块 + 单测（单进程）；真机联机（R4-V）另见 RUNNER-HANDOFF。
> 位置：src\qqbot\；venv = .venv-qqbot（Python 3.10.2）；测试根 = src\qqbot（pytest.ini）。

### 27.1 八模块与单向依赖（D-029：九改八，binding 并入 ledger）
- adapter —— WS 收发 / echo 配对 / 重连，唯一接触 OneBot 协议。
- parser   —— 文法（纯函数，零 IO）。
- resolver —— 目标 → 坐标：城市缓存 + 对账、军队现取。
- ledger   —— 点数 / 冻结 / 退款 / 冷却 / 绑定表（bindings 并入，D-029）。
- catalog  —— 商品表（价格 / 冷却 / 是否待批 / 文案）。
- executor —— 唯一桥接出口（只允许它调用 urllib），串行队列 + 30 s 超时。
- receipt  —— 两段式回执、延迟结算。
- approval —— 待批队列（容量 5 / 60 s 超时 / 三条退款路径）。
- 依赖单向无环，全由 bot.py 正向装配。
- ★ executor 唯一 HTTP 出口 grep 闸门：
  `grep -rn "urlopen" src\qqbot\` 只许命中 executor.py；
  更严：`grep -rnE "urlopen|requests\.|httpx\.|http\.client|aiohttp" src\qqbot\` 也只许命中 executor.py。

### 27.2 NapCat 接入（D-024，双模式，默认反向）
- 默认反向：bot 作 WS 服务端，监听 127.0.0.1:6199，路径 /ws；NapCat 侧 websocketClients
  enable=true、url=ws://localhost:6199/ws——本机现配置即此，零改动（无 websocketServers 项）。
- forward 可选：NapCat websocketServers（127.0.0.1:3001），bot 作客户端；config 切换。
- ★ messagePostFormat="array"：需按段解析 @ 与文本，string 格式会把 CQ 码混进正文。
- ★ reportSelfMessage=false：防止 bot 自己的回执被当成新指令（自激环）。
- heartInterval=30000 心跳；反向模式 bot 若重启，NapCat 按 reconnectInterval=30000
  最长 30 秒才重连，开发期需容忍。
- 与 NoneBot2 对比：本项目不上 NoneBot，理由不同（D-024：复用现配置、零改动）。

### 27.3 websockets 16.1.1 API 差异（R4-2c 实测；照抄 HANDOFF-R4 §1，最易踩坑）
[联网核实 2026-09-19，官方 upgrade 文档 stable 版]
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
- ★ 细节实装时以官方 upgrade 文档复核，勿凭记忆硬写。

### 27.4 pytest 配置（pytest-asyncio 1.4.0 strict 模式）
默认 asyncio_mode = strict：异步测试不加标记会被跳过或报错；
asyncio_default_fixture_loop_scope 未设则刷警告。
故 src\qqbot\pytest.ini 写三项：asyncio_mode = auto、loop_scope = function、testpaths = tests。
异步测试直接 async def test_ 即可，勿加 @pytest.mark.asyncio。
跑法：从项目根（含 src\qqbot 与 .venv-qqbot 的目录）执行 `python -m pytest src\qqbot -q`。

### 27.5 C1~C6 六条硬约束 → 模块 / 函数 / 测试（防日后误删防呆）
| 硬约束 | 落点模块.函数 | 对应测试 |
|---|---|---|
| C1 打击两段式回执 | receipt.ack_attack（受理）+ receipt.settle_attack（延迟结算） | tests/test_receipt.py::test_d_two_phase_ordering |
| C2 播报前先判 max_warriors==0 | broadcaster.render_situation | tests/test_broadcaster.py::test_j_max_warriors_zero_no_ratio |
| C3 warrior_slots/max 只作「上一周期快照」 | broadcaster.render_situation | tests/test_broadcaster.py::test_j_max_warriors_positive_shows_ratio |
| C4 不用 is_alive 判城毁 | resolver.resolve_city（只看是否在列表中；is_alive 仅记录） | tests/test_resolver.py::test_d_is_alive_false_but_present_live |
| C5 与 8723 count 对账 | resolver._refresh_cities（退避重拉 ≤3 次；未过一律 UNSTABLE） | tests/test_resolver.py::test_b_reconcile_mismatch_then_match / test_c_unstable_wins_over_city_gone |
| C6 非暂停态投点前重取坐标 | resolver 新鲜度机制（军队每次现取不缓存；城市缓存 TTL 10 s，_cached 到期即重拉） | tests/test_resolver.py::test_a_ttl_cache_hit_then_refetch |
★ C6 真正「invoke 前 1 s 内重取并投点」的强制，落在联机写路径（R4-V：绑定目标值后、
  invoke 前重取）；离线骨架只搭了新鲜度脚手架，勿误删它的缓存/对账逻辑。

### 27.6 ledger TOCTOU 教训（R4-2b）
hold_attack / hold_build 的「查冷却 → hold → 写冷却」三步若各自 _txn 独立，锁会在步间
释放 → 并发可绕过个人 60 s 冷却。修法与判据：
- _txn 必须可重入：threading.RLock + 嵌套深度计数，深度 0 才 BEGIN、回 0 才 COMMIT，
  不用 SAVEPOINT。
- 组合操作「查 + 改」必须用最外层 _txn 包住全过程，在同一事务内完成。
★ 反向验证过：退回普通 Lock（不可重入）后，5 并发 hold_build 有 2 个绕过 60 s 冷却
  ——不是理论风险，是实测窗口。

### 27.7 踩坑实录（本机环境）
1. 127.0.0.1:17892 回环代理会破坏 schannel TLS，导致 git push 到 github 失败；
   遇 push 失败先排除该代理（走直连 / 无代理绕过）。
2. PowerShell 5.1 读无 BOM 的 .ps1 会按 GBK 解释，中文脚本解析崩溃；
   写 .ps1 用 UTF-8 with BOM，或避免在脚本里内嵌中文字面量。
3. PowerShell 下 grep 正则的转义层数与 bash 不同——把形如 `<盘符>:\...` 的用户目录
   反斜杠路径直接写进正则，会因多/少一层转义而静默 0 命中；本机曾因此误判 0 命中。
   核查路径类 grep 时改用单引号字符串包裹，或改用 Python re（case-sensitive、转义可控）。
