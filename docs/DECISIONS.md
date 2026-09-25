# DECISIONS — 决策与排除项（ADR 风格，只增不删）

> 始建于：2026-09-14（任务书 2026-09-14-A，从 WORLDBOX-PROJECT-STATE.md 第六节迁移并补全）
> 格式：每条 = 日期 / 状态（有效 | 已推翻）/ 背景 / 决策 / 理由 / 证据 / 推翻条件

---

## D-001 放弃破解版旧版本路线，改用 0.51.2 学习版
- 日期：2026-09（项目早期，具体日期失记，归档于 09-14）
- 状态：有效
- 背景：项目起步时曾考虑 0.13/0.22.9 等旧版本路线，以匹配曾流传的"开盘辅助 mod"等工具。
- 决策：放弃旧版本路线，用户换到 0.51.2 学习版，此路径才打通。
- 理由：worldbox-mcp 桥接靠反射解析 0.51 的类结构，跨版本差异过大；NCMS 已停维护（仅支持到 0.21.1）；
  0.22 时代的"开盘辅助 mod"（B 站 @贝伦帝国 等）在 0.51.2 上不可用。
- 证据：Claude 侧的版本资料调研（具体出处未入档）。
- 推翻条件：若目标工具链的版本支持矩阵发生变化（如某关键桥接明确支持旧版本），可重新评估。

## D-002 排除 MultiplayerBox 联机方案
- 日期：2026-09（归档于 09-14）
- 状态：有效
- 背景：项目目标 3 需要多人远程操作，曾考虑"官方直连 + 社区联机 mod"路线。
- 决策：排除社区 MultiplayerBox（GameBanana 656965），改为 QQ bot + AI GM 方案。
- 理由：官方永不支持联机（开发者 FAQ 明确否定）；MultiplayerBox 数据惨淡（8 赞/17 订阅）、
  未标明支持 0.51.2、口碑差。
- 证据：开发者 FAQ（社区说法）；GameBanana 页面数据（Claude 侧查阅）。
- 推翻条件：若出现官方联机能力，或 MultiplayerBox 针对 0.51.2 发布稳定版并被实测验证，可重新评估。

## D-003 群友不用 worldbox-mcp 的 faction_player 角色，改用 God token + 自实现权限
- 日期：2026-09（归档于 09-14）
- 状态：**已推翻（2026-09-14 E 轮）**
- 背景：worldbox-mcp v0.3+ 提供多 agent 角色系统（god/faction_player/observer/narrator），
  初看适合给群友分角色。
- 决策（原）：QQ bot 独占一个 God token；权限/配额/冷却在 Python 侧自己实现。
- 理由（原）：faction_player 角色实际只剩 spawn 一个动作（invoke_power 和 paint_tile 因全图生效
  被锁在 God 角色），群友会很快腻。
- 证据（原）：worldbox-mcp docs/multi-agent.md 的权限矩阵（Claude 侧审阅）。
- **推翻原因（E 轮）**：上述限制是 **0.5.0 的 breaking change** 才引入的
  （0.5.0 起 faction_player 调用 invoke_power 返回 PERMISSION_DENIED）；
  本项目钉的是 **0.4.0**，其 faction_player 同时拥有 spawn 与 invoke_power
  （ActionFaction 权限组），并且还拥有 AdvanceTime（pause/resume/set_speed）。
  ★ 附带结论：选 0.4.0 意外保住了群友玩法的核心可玩性，
  这是 D-008（钉 0.4.0）此前未被认识到的额外收益。
  ★ 重新评估方向：session 层原生提供 per-agent token、角色权限门、fog-of-war 读过滤、
  per-agent 消息总线、turn-order 可选——部分能力无需在 Python 侧重写；
  但速率限制/配额/冷却仍无原生支持，需在 Python 侧自行实现。
- 推翻条件：已触发（E 轮依据 0.4.0 实测确认推翻）；若未来升级 mod 至 0.5.0+ 则旧限制回归。

## D-004 QQ 侧不走 MCP，直连桥接 HTTP
- 日期：2026-09（归档于 09-14）
- 状态：有效
- 背景：桥接同时提供 MCP（给 LLM 客户端）与 HTTP /cmd 接口。
- 决策：QQ bot 是自写程序，直接 POST /cmd 打桥接 HTTP 接口；LLM 用原生 function calling。
  MCP 只留给用户在 Cherry Studio 里当运维台（看状态、救火、手动干预）。
- 理由：MCP 是给 LLM 客户端用的协议；自写程序走 MCP 是无谓的协议套利。
- 证据：worldbox-mcp README 架构图与设计说明（审阅存档于 downloads\mcp-bridge\README.md）。
- 推翻条件：若 QQ bot 侧最终用现成 LLM 客户端框架且其原生支持 MCP，可重新评估。

## D-005 外交用"叙事+后果"实现，不驱动 mod UI
- 日期：2026-09（归档于 09-14）
- 状态：有效
- 背景：曾想通过桥接点击 EmpireCraft 的外交/结盟按钮。
- 决策：由 AI GM 用叙事+后果实现外交——要结盟就停止边境刷怪、给盟友刷援军；
  要宣战就在边境刷兵制造冲突。
- 理由：桥接的 click_special_action 尚无人实现（invoke_power 覆盖不了按钮点击）；
  WorldBox 的战争本就由游戏 AI 自行推进，GM 只需注入扰动。
- 证据：worldbox-mcp README 贡献区明确列出 click_special_action 为待实现项。
- 推翻条件：若 click_special_action 被实现且 EmpireCraft 的按钮经实测可被驱动，则可改为直接驱动 UI。

## D-006 不混装 BepInEx 与 NML 后再排错
- 日期：2026-09-13/14（归档于 09-14）
- 状态：有效
- 背景：两套独立注入机制（NML 走 NCMS 兼容层，BepInEx 走 doorstop），出问题时难以定位。
- 决策：必须先单独验证 NML 通过（已完成，2026-09-14），再装 BepInEx。
- 理由：分步验证可明确故障归属。
- 证据：NML+EmpireCraft 已实测通过（编译零报错、100+ 年正常、存读档正常）；
  回滚快照已备（snapshots\snapshot-working-nml-empirecraft\）。
- 推翻条件：无（这是流程性决策）。若 BepInEx 安装后验证通过，此决策使命完成归档。

## D-007 不使用官方 install-mod.ps1，改为手工安装
- 日期：2026-09-14
- 状态：有效
- 背景：官方 README 的一行安装命令 `iex (irm ...install-mod.ps1)` 存在两个不可接受点。
- 决策：按其逻辑手工执行安装，全程可控；脚本原文已存档审阅，只下载不执行。
- 理由：① 脚本含 Stop-Process -Force 强杀游戏（有写坏存档风险，违反协作约定）；
  ② 下载 BepInEx 与 WorldBoxBridge 均不校验哈希。
- 证据：install.ps1 全文审阅实录（存档于 downloads\mcp-bridge\install.ps1，7,331 B，
  SHA256 0F45918BD4CA6D7ABB4CA7C615B3DD98410C2B1C2E2C47241328F539D6AF3DAC）：
  - Stop-Process -Force 位于 Stop-WorldBoxIfRunning 函数
  - Download-To 为裸 Invoke-WebRequest，无任何哈希校验
  - 自动发现逻辑：硬编码 2 条 Steam 路径 + 全盘扫 X:\SteamLibrary / X:\Steam / X:\GAMES ——
    不含 E:\game\worldbox；脚本支持 -WorldBoxPath 显式参数
  - 写入目标：游戏根 BepInEx\*、winhttp.dll、doorstop_config.ini、plugins\WorldBoxBridge.dll、
    config\WorldBoxBridge.cfg；不碰 StreamingAssets\mods
  - 无注册表/计划任务/环境变量/防火墙/遥测行为；仅 HTTP GET 下载
  - token：48 位随机 alnum，存 config\WorldBoxBridge.cfg，已存在则保留
- 推翻条件：官方脚本补上哈希校验并改为优雅关游戏后，可重新评估使用脚本一键装。

## D-008 WorldBoxBridge 选 0.4.0 而非最新版
- 日期：2026-09-14
- 状态：有效
- 背景：下载时最新版为 0.6.0；作者 compatibility.md 显示不同版本验证状态差异大。
- 决策：安装 v0.4.0（已弃用已下载的 0.6.0，移入 deprecated 留档）。
- 理由：0.4.0 是作者 compatibility.md 中唯一 ✅ 真机验证的版本；
  0.5.0/0.6.0 为 🔵 未经真机验证；0.3.0–0.3.3 曾发布无法加载的 DLL。
- 证据：实测 0.4.0 加载成功、/health 与 /capabilities 均 200、28 commands；
  0.6.0 未实测，仅文档状态不佳（Claude 侧查阅 compatibility.md）。
- 推翻条件：0.5.0 或更高版本在矩阵中转为 ✅，或本地链路稳定后主动升级验证。

## D-009 BepInEx 选 5.4.23.5 而非 5.4.23.2
- 日期：2026-09-14
- 状态：有效
- 背景：最初任务书指定 5.4.23.2（2024-06），后发现其无官方哈希可校验。
- 决策：改用 5.4.23.5（2026-02-08 发布）。
- 理由：5.4.23.2 在 GitHub API 中 digest 字段为 null（资产摘要功能 2025-06 才上线且不回溯），
  无官方哈希可对；5.4.23.5 有官方 digest，且与作者 ✅ 验证记录中使用的版本一致。
- 证据：三方比对通过（API digest = 实测 SHA256 = 任务书值 82f98785…）；
  5.4.23.2 zip 已移入 deprecated 留档。
- 推翻条件：若 5.4.23.5 出现已知缺陷而 5.4.23.2 可从其他可信渠道取得哈希。

## D-010 玩法定位由回合制 GM 改为投屏 + 即时战略（RTS）
- 日期：2026-09-14
- 状态：有效
- 背景：交接文档第七节长期记录「回合制推进（收齐请求→暂停→执行→快进→暂停→截图发战报）」，
  实为已被用户放弃的旧方案；文档过时导致 Claude 重置后反复朝错误方向设计。
- 决策：终局玩法 = 投屏 + 群友共同参与的 RTS；世界持续运行；观众经投屏画面观察；
  GM 不推送战报；群友各自远程操作一个国家 PvP/PvE；用户降格为普通玩家，
  GM 由 QQ bot + AI agent 担任。
- 理由：用户明确的终局玩法定位（2026-09-14 拍板）。
- 证据：用户指示（任务书 2026-09-14-C 第 1 节）。
- 影响面：取消战报推送需求；取消回合闸门设计；并发指令与限流需重新设计。
- 推翻条件：用户再次变更玩法定位。
- P 轮修正：场景为投屏给 ≤5 位朋友，无公开观众压力，崩溃可接受可重开；结论不变，仅理由描述更新。

## D-011 Python MCP server 钉 0.4.0 与 mod 版本对齐
- 日期：2026-09-14
- 状态：有效
- 背景：PyPI 最新 0.6.0；mod 侧 WorldBoxBridge 钉 0.4.0（D-008）。0.7.0 只有 GitHub tag
  未发布到 PyPI（Claude 侧曾误报"PyPI 已到 0.7.0"，已更正）。
- 决策：venv 显式安装 worldbox-mcp==0.4.0（E:\work\worldbox-ai\src\mcp-server\），
  禁止裸装 latest。
- 理由：跨版本协议不匹配风险（0.6.0 server 可能向 0.4.0 mod 发新字段/端点）。
- 证据：0.4.0 server + 0.4.0 mod 实测 stdio 冒烟通过（29 工具、
  capabilities 与基线一致、EmpireCraft power 资产可见）。
- 推翻条件：mod 侧升级时同步升级 server 并重跑冒烟。

## D-012 测试世界与快照分离归档
- 日期：2026-09-14
- 状态：有效
- 背景：snapshots\ 的语义是「某个已验证安装配置的完整镜像」；把测试存档混入快照
  会导致每次快照的文件数与体积漂移——本项目已因度量基线不干净发生过两次对账困难
  （1578 vs 1595 文件、59.7 vs 53.1 MB）。
- 决策：新建 testworlds\ 存放可重复实验用的基准世界；snapshots\ 只存安装配置镜像。
- 理由：职责分离，度量基线干净；测试档可独立迭代而不污染回滚点。
- 证据：2026-09-14 E 轮起 testworlds\ 已有 year102 手动档 + year121 自动档
  （12 文件 8.5 MB，附 README 记录年份/来源/复制日期/SHA256）。
- 推翻条件：无（职责分离原则，除非项目不再需要快照对账）。

## D-013 投屏场景下不启用战争迷雾（partial_intel: false）
- 日期：2026-09-14
- 状态：有效
- 背景：partial_intel 是 API 侧的读取过滤，不是渲染层遮挡；WorldBox 画面本身始终是
  上帝视角全图。用户在自己电脑上投屏，观众与玩家都能从画面看到全部信息，
  因此投屏流本身就是一条绕过迷雾的旁路（作者堵掉了 screenshot 这个小洞，但推流是正门）。
- 决策：投屏场景下 partial_intel 保持 false（不启用迷雾）。
- 理由：强行开启有三大坏处——①迷雾形同虚设（画面全可见）；②玩家 AI 查不到敌国城市
  只能瞎打，等于逼人"用眼睛作弊、用 API 装瞎"；③partial_intel 为 true 时
  faction_player 不能截图，废掉让视觉模型看画面干活这条路。
- 推荐配置：scenario = hierarchical；用户/GM 为 god 角色；朋友各为 faction_player；
  partial_intel: false；turn_based: false。
- 推翻条件：若将来改为不投屏、或玩家改为完全依赖 AI 而不看画面。
- P 轮修正：场景为投屏给 ≤5 位朋友，无公开观众压力，崩溃可接受可重开；结论不变，仅理由描述更新。

## D-014 将来启用 agents.json 时，必须为用户保留 god 角色 agent
- 日期：2026-09-14
- 状态：有效（仅落档，未执行）
- 背景：agents.json 一旦存在，bridge 立即退出 legacy 模式；legacy 为
  id="legacy"、role=God、全权限、token 取自 WorldBoxBridge.cfg。
- 决策：将来启用 agents.json 时，必须在其中为用户自己保留一个 role=god 的 agent，
  且其 token 直接沿用 WorldBoxBridge.cfg 中现有的值，使 Cherry Studio 运维台
  无需改配置即可继续工作。
- 理由：否则现有运维台 token 会当场失效（legacy 模式退出后 cfg token 不再被承认）。
- 证据：0.4.0 文档原文（agents.json 存在即退出 legacy；上游文档 2026-09-14 E 轮核对）。
- 推翻条件：若运维台改用独立新 token 并接受重新配置成本，则本决策可简化。
## D-015 取消虚拟国库，改用游戏真实资源
- 日期：2026-09-14
- 状态：有效
- 背景：早期因 list_kingdoms / list_cities 不返回资源字段，曾计划在 Python 侧做虚拟账本。
- 决策：改用游戏真实资源读写，取消虚拟国库。
- 理由：M 轮反编译实证 CityResources.get/change/set 全 public，可直接读写真实资源；
  虚拟账本会与游戏内实际状态长期漂移，且玩家在直播画面能看到真实数字，
  两套数据对不上会破坏可信度。
- 证据：FEASIBILITY 段 3.2；list_kingdoms 返回 capital_id 使扣款目标城市可定位（O 轮实测）。
- 推翻条件：若真实资源写入被证实导致游戏状态异常。

## D-016 走独立 BepInEx 插件，不改 WorldBoxBridge 源码
- 日期：2026-09-14
- 状态：有效
- 背景：资源扣减与强制宣战/停战都不在 0.4.0 的 28 条命令内，必须自写。
- 决策：新建独立 BepInEx 插件、独立端口，WorldBoxBridge 0.4.0 保持原版不动。
- 理由：① 0.4.0 是作者唯一真机验证版本（D-008），改源码等于丢掉这个保障；
  ② 独立插件出问题只需删一个 dll，回滚成本最低；
  ③ 两者可并存加载（BepInEx 插件机制本身支持）。
- 代价：鉴权与 HTTP 管道需自实现；可照抄 worldbox-mcp（MIT）的
  MainThreadDispatcher 与 GameRefs 模式。
- 证据：FEASIBILITY 段 1.5 模板齐备，O 轮编译空壳已验证链路（0 错误）。
- 推翻条件：若并存加载实测失败。

## D-017 军队调度参照 EmpireCraft 前线实现，不直写目标字段
- 日期：2026-09-14
- 状态：有效（方向已定，尚未实现）
- 背景：O 轮实证 City.target_attack_city 与 Actor.beh_tile_target 存在，
  但由 CityBehCheckAttackZone.execute() 每行为周期重置重派，直写不持久。
  P 轮发现 EmpireCraft 已实现同类机制（ActorExtension.SetFrontLineMoveTarget
  等 7、8 个方法 + EmpireCraftActorCheckWarriorMove 自写 AI 行为 +
  HighPopulationPerformance 帧缓存），且该实现正在本机栈内稳定运行。
- 决策：自写调兵命令时以 EmpireCraft 前线实现为参考架构
  （自维护目标表 + AI 行为层介入 + 帧缓存），不直接写游戏目标字段。
  是否反射复用 EmpireCraft 的扩展方法另行决策（见代价）。
- 理由：直写字段必被下一周期覆盖；EmpireCraft 的实现是同版本、
  同环境、已验证可用的现成参考，比从零设计可靠。
- 代价：若选择反射复用，将硬依赖 EmpireCraft（NML 运行时 Roslyn 编译产物、
  程序集名含中文、加载顺序不可控、mod 更新即可能失效）；
  若选择自实现，需重做目标表与 AI 介入，工作量大但无外部依赖。
- 许可证前置条件：许可证状态存疑——仓库 LICENSE 逐字为 MIT 但版权行为 Copyright (c) 2023
  WorldBoxOpenMods（NeoModLoader 组织，疑模板残留）；GameBanana mod 602979 标 CC BY-NC-ND 4.0。
  仅本地私用不分发；优先照抄架构思路而非源码。（Claude 侧独立核实：GitHub 仓库根 LICENSE
  与 GameBanana 页面，2026-09-16）
- 证据：P 轮 0-1 命中清单与 4B 源码研读；O 轮 isOkToSendArmy（0.7f）
  与 CityBehCheckAttackZone.execute() 重置逻辑。
- 推翻条件：若 4B 研读表明该实现依赖 EmpireCraft 特有数据结构而无法移植。

## D-018 一次性娱乐项目，不计长期可维护性
- 日期：2026-09-14
- 状态：已定
- 背景：本项目为朋友间娱乐（≤5 人），非生产环境。
- 决定：不为版本 / mod 兼容性付出额外成本；允许硬依赖第三方 mod、
  反射 internal 成员、照搬第三方架构（均在本地验证后使用）。
- 理由：一次性娱乐用途，崩溃可接受可重开；存档有自动保存可回退。
- 推翻条件：若项目转为长期维护。

## D-019 自写插件放弃「无游戏类型引用」隔离，改直接引用游戏程序集
- 日期：2026-09-16
- 状态：有效
- 背景：list_armies 需要枚举 World.world.armies 并读 Army/City/Kingdom 等类型，
  反射缓存写法（照 0.4.0 WorldAccess）代码量大、可读性差。
- 决策：WBAIBridge 0.2.0 起直接引用 Assembly-CSharp.dll + Newtonsoft.Json.dll（13.0.2），
  均为 Private=false 只读引用、不复制不修改。
- 理由：按 D-018 一次性娱乐项目，不计长期可维护性，用代码量与复杂度下降换稳定性余量。
- 影响面：插件从「不可能影响游戏」变为「游戏版本一变即可能类型加载失败」；
  缓解：命令外层 + Execute 内层双层 try/catch，失败仅禁用 list_armies，/health 与插件本体不受影响。
- 回退条件：删 plugins\WBAIBridge.dll 即回滚到 0.1.0（backups\WBAIBridge-0.1.0-abe4e241.dll）。

## D-020 0.7f 出兵线不作为玩法枢纽
- 日期：2026-09-16
- 状态：有效
- 背景：曾拟以 isOkToSendArmy 阈值（0.7f）为核心循环（决定 AI 何时出兵）。
- 决策：0.7f 仅作战况播报的只读指标，不作为可操作杠杆。
- 理由：四条杠杆全部实测报废——打平民延缓出兵（方向相反，静态证实分母随人口走）、
  打平民逼出兵断补员（R2-8c 分母未塌缩，人口反升 641→658）、
  盖房钉兵（R2-8b 未成立）、打军队压分子（R2-9a 3 发减员 1，效率过低）。
  根因：attacker 槽位每周期步长 2 且与 _citizens_left 取小，对人口小幅扰动极迟钝。
- 反转条件：若自写命令能直接写 warrior_slots 或批量改人口，重新评估。

## D-021 打击类降级为天灾，主轴改为目标指派
- 日期：2026-09-16
- 状态：有效
- 依据：打击类对军队实测近乎无效（R2-9a），神力不是有效战斗手段。
- 决策：打击类定位为战场变数 / PvE 天灾；RTS 主轴为 D-017 目标指派；
  pause/set_speed 作为战斗节奏控制器（解决 QQ 消息往返延迟）。
  插件优先级改为：list_cities_ex → 目标指派（D-017）→ 资源读写。
  资源读写后移不影响 v1（v1 以时间为货币）。

## D-022 打击类按目标形态分工
- 日期：2026-09-17
- 状态：有效
- 依据：lightning 打城市中心 -4、meteorite 同点 -180（R3-V-6，精确中心投点实测）。
- 决策：lightning = 野外单位点杀（含统帅）；meteorite = 城市洗地，
  高价 + 长城市冷却；两者非价格档位差异而是功能差异。
- 反转条件：若后续发现居民出户时段 lightning 对城市有效，重新评估。

## D-023 QQ bot v1 消息链路插队至目标指派（D-017）之前
- 日期：2026-09-17
- 决策：先做 QQ bot v1 消息链路，再做目标指派（D-017），最后资源读写。
- 理由：v1 为纯 Python，离线可开发可单测，不占用户在场开机的轮次；
  D-021 的插件内部优先级不变（list_cities_ex 已完成 → 目标指派 → 资源读写）。
- 影响：D-017 静态可行性轮次后移；v1 以时间为货币（真实资源扣款留在 v2，D-015）。
- 回退条件：若 v1 链路证明离线不可单测、必须反复开机联调，则回到先做 D-017。

## D-024 QQ bot 接入 NapCat 采用双模式适配，默认反向 WebSocket
- 日期：2026-09-17
- 决策：adapter 同时支持反向 WS（bot 作 WS 服务端）与正向 WS（bot 作客户端），
  由 config 开关切换；v1 默认反向，复用本机现有配置
  ws://localhost:6199/ws（messagePostFormat=array、reportSelfMessage=false）。
- 理由：R4-0 实测本机 websocketServers 为空、websocketClients 已有一条 enable=true
  的可用反向配置，且 array 与 reportSelfMessage 参数正好符合需求；
  改走正向需先启 NapCat、扫码登录并在 WebUI 新建配置（扫码会刷新 WebUI token
  并强制改密码），为一个次要优点付 GUI 代价不值。
  ★ 与 NoneBot2 官方"反向 WS 推荐"方向一致，但本项目不使用 NoneBot，
  推荐理由不同（此处理由是复用现配置、零改动）。
- 影响：bot 需监听 6199；bot 重启后 NapCat 按 reconnectInterval=30000
  最长 30 秒才重连，开发期需容忍。
  ★ 已知冲突风险：6199 是 AstrBot 文档中反向 WS 的默认端口，现有配置名为 "test"，
  疑为旧 AstrBot 部署遗留；若日后重启 AstrBot 会撞端口。
- 回退条件：若 30 秒空窗影响联调，改 reconnectInterval=5000 或切正向模式
  （NapCat 侧新建 websocketServers），改动限于 config 与 adapter 建连部分。

## D-025 QQ bot v1 依赖最小化：config 用 JSON，不引入 TOML/HTTP/异步 SQLite 库
- 日期：2026-09-17
- 决策：配置文件用 JSON（stdlib json）；HTTP 调用用 stdlib urllib.request
  置于 asyncio.to_thread；账本用 stdlib sqlite3 同法；
  第三方依赖只有 websockets、pytest、pytest-asyncio。
- 理由：本机 Python 3.10.2 无 tomllib（3.11 才进标准库），为配置格式引入
  tomli 不值；executor 串行 + 30 秒超时，stdlib 客户端已够用。
- 影响：QQBOT-V1-ARCH 十节所写 config.example.toml 改为 config.example.json，
  属实现偏差，记入 SESSION-LOG，设计稿正文本轮不改。
- 回退条件：若后续需要连接池/HTTP2/高频并发，再引入 httpx 并单独记一条决策。

## D-026 meteorite 进待批队列（Q3 拍板）
- 日期：2026-09-19
- 状态：有效
- 决策：meteorite 加入待批队列（一人一令一码、60 秒超时，放行后执行）；
  lightning 保持 needs_approval=false。
- 理由：12 点约等于攒 2 小时，一条群消息即不可逆的城市级打击，60 秒反悔窗口成本
  远低于误伤代价。lightning 保持 needs_approval=false（3 点，且打城市另有二次确认）。
- 影响面：catalog 默认值（meteorite.needs_approval=true，R4-2d 已改）、approval 路径、
  R4-V 演示流程。可回退（改一个布尔值）。

## D-027 费率与价目定稿（Q2 拍板）
- 日期：2026-09-19
- 状态：有效
- 决策：攒点 10 分钟 1 点 / 上限 30 / 新人 5；lightning 3；meteorite 12 + 城市冷却
  1800 秒；七种墙各 1；living_house 1；建设类个人冷却 60 秒。
- 理由：Q2 拍板；v1 以时间为货币（D-015 / D-023）。
- 影响面：已写入 config.example.json 默认值（R4-2d），catalog 与 config 保持一致。

## D-028 BridgeResult 增加 body 只读字段
- 日期：2026-09-19
- 状态：有效
- 决策：executor 的 BridgeResult 增加 body 只读字段。三约束：① 解析失败 body=None，
  且不影响 status 判定；② digest 仍是写 ledger.bridge_resp_digest 的唯一来源，
  body 禁止落库 / 进 ledger_log；③ body 超 64 KB 置 None。
- 理由：resolver 需读桥接结构化数据，原设计只返摘要是参谋侧疏漏（摘要供账本审计，
  非供业务读数）。否决了「另开 only-data 通道」方案，因其在「HTTP 只走 executor」
  闸门上凿洞。
- 影响面：executor.py（BridgeResult / _classify）、resolver / receipt / broadcaster 读
  body；R4-2d 已实装并通过 D-0 裁决要求的三条约束测试。

## D-029 binding 并入 ledger，模块数由九改八
- 日期：2026-09-19
- 状态：有效
- 决策：binding 不再是独立模块，并入 ledger；ARCH §2 的九模块划分据此修订为八模块。
- 理由：绑定表与点数表同处一个 sqlite，硬拆成独立模块需跨模块开事务。
- 影响面：ARCH §2 模块表（binding 行并入 ledger 行）；ledger 提供
  bind / rebind / mark_dead / get_binding；R4-2d 装配与 R4-3 文档同步。

## D-030 v1 待批队列不持久化（已知缺口，非遗漏）
- 日期：2026-09-19
- 状态：有效
- 决策：v1 待批队列不持久化；ledger.pending_approval 表已建但未启用，
  approval 为进程内内存队列。
- 理由：避免持久化复杂度，v1 暂以内存队列承载（容量 5 / 时限 60 / 退款三条路径）。
- 影响面：后果——bot 在待批中途重启，在队申请连同已冻结的点一起蒸发，既没退也没花。
  ★ R4-V 联机时不得在待批中途重启 bot。持久化留待 v1.1。

## D-031 R4-V 验收期初始点数按 30 接线
- 日期：2026-09-22
- 状态：有效
- 背景：D-027 定"新人 5 点"，config.local.json 实为 point_start=30，
  而 ledger.py:111 initial 默认 5 且装配处 bot.py:168-170 三个键一个都没接线，
  实际生效值恒为 5（J-1.7 实测，points 表 0 行故无历史污染）。
- 决策：point_start 取 30，并在 bot.py 装配处显式接线 initial / cap /
  accrual_interval_sec 三项；配置缺失或非正数时启动即失败，禁止回落默认值。
- 理由：D-027 的 meteorite 定价 12 点、攒点 10 分钟 1 点，新人 5 点需 70 分钟
  才够一发，R4-V 验收段走不下去。30 = 上限值，等于"验收期直接给满"。
- 影响面：D-027 的"新人 5"在 R4-V 验收期内以本条为准；验收结束后是否回归 5
  需另立决策。ledger.py 不改，仅改装配接线。
- 证据：J-1.7 实测（config 三键值、ledger.py:111-113、bot.py:168-170、points 行数 0）。
- 推翻条件：R4-V 验收结束，或改用真实资源扣款（D-028/D4）。

## D-032 own 目标语义（作用域与名称拆参）
- 日期：2026-09-23
- 状态：有效
- 背景：B-16 暴露 _resolve_coords 把 binding.kingdom_id（字符串 "17"）当城名
  喂给按 startswith 匹配的 resolve_city，属作用域参数与名称参数混用。
- 决策：作用域与名称拆成两个参数。
  a) 我国 <城名>：先按 kingdom_id 过滤，再在该国范围内按城名前缀匹配；
     命中多个走 ARCH §3.4 候选列表，不猜。
  b) 我城（不带城名）：该国城数 == 1 直接命中；> 1 回候选列表要求重发。
     禁止默认取首城或 capital——.造 写持久数据，猜错不可逆。
  c) 我国首都：本段只读确认桥侧是否存在 capital / is_capital 字段；
     无字段则登记缺陷、v1 不上该形态，解析失败按「看不懂这个目标」处理。
- 理由：ARCH §3.3 目标文法与 §3.4 歧义处理已规定不猜；A 档写操作不可逆。
- 更新（2026-09-23，Y-0）：**第三条解除阻塞** —— 8723 list_kingdoms 实测含 `capital_id`
  字段（kingdom 22→3、kingdom 7→7、kingdom 24→1）⇒「我国首都」形态**可实装**：
  取绑定国的 capital_id，再在 list_cities_ex 中按 id 定位；capital_id 所指城不在列表中时
  按 UNSTABLE 处理（先过 C5 对账），不得判 CITY_GONE。原「无字段则不上该形态」的退路作废。
- 推翻条件：桥侧出现权威首都字段，或 §3.4 交互被改版。

## D-033 世界运行判据（以 tick 差值为准）【已作废 · 见 D-040】
- 日期：2026-09-23
- 状态：**【已作废】（2026-09-25 裁决；D-033 存废 → 见 D-040）**；原条保留不删，仅留追溯。
- 作废依据：Z-3 全程观测（本地 `e64439435124fb24089c674341640b26aa622533` /
  镜像 `35c4513686f8fb1bf528835c87e5fc20465f8260`）证明 tick 为帧计数而非世界时间。
- 【原条原文】决策：世界是否在跑，一律以 tick 差值为准；get_world_state.paused 字段不作判据。
- 理由：B-18 实测该字段与 resume.previous_paused 及 tick 速率相互矛盾。
- 影响面（原条）：ARCH §5.2 的 C6（非暂停态须在 invoke 前 1 秒内重取坐标）实装时不得读
  paused，必须改用 tick 差值，否则会在「实际在跑但字段说暂停」时跳过重取而打偏。
  打军队目标前必须闭合。
- ★影响面（作废后更正）：C6 **不得改用 tick 差值**（tick 亦非世界时间），须按 D-040 的判据集处理。

## D-034 改世界动作验收期的世界默认态
- 日期：2026-09-23
- 状态：有效
- 决策：V 段验收期间世界默认保持暂停；resume 仅在明确需要世界推进时执行，
  且执行时必须同时约定回暂停的时刻或条件，由执行侧自行落实。
- 理由：本轮 00:52 resume 后无回暂停约定，世界连跑约 12 小时，kingdom 17 灭亡，
  V-3 全部基线作废、绑定失效（见 SESSION-LOG W-1.7）。
- 依据：QQBOT-DESIGN 5.2 实测「暂停态下桥接完全可用，不影响 /cmd 与播报」，
  暂停不妨碍 .造 / .打 的验收。
- 推翻条件：需要观测世界自演化的验收项（如国灭播报）时，按项单独 resume。

## D-035 重绑至 kingdom 20（★已推翻，见 D-039）
- 日期：2026-09-23
- 状态：**已推翻（2026-09-23，Y-0）**
- 背景：绑定 kingdom_id="17" 所指王国已不存在（KINGDOM17_CITIES=0）。
- 原决策：defect E 修复并通过校验后，将唯一绑定改绑 kingdom 20「辉腾衮  叛乱」。
- 推翻原因：kingdom 20 已灭；13:39 时城 1-6/8 归 kingdom 22「清泉 帝国」，
  18:33 时 kingdom 22 已改名「白鹿 帝国」且城数由 7 降为 6，另生成 kingdom 24。
- 替代：见 D-039。
- 影响面（保留有效部分）：.国情 验收基准需按 D-039/Y-3.5 在冻结态重取；旧基准作废。

## D-036 国灭重绑免批
- 日期：2026-09-23
- 状态：有效
- 决策：绑定所指王国不存在时，该绑定置 dead，持有者可免批、免点数重绑一次；
  王国仍存在时的普通重绑维持 §3.2「需 God 批」，本段不实装该审批流。
- 理由：ARCH §4.2 原文已授权此路径；kingdom 17 已灭（SESSION-LOG W-1.7），
  若不走此路径，则只能在「无法重绑」与「无校验裸重绑」之间二选一。
- 影响面：解除「defect E 修好前禁止任何重绑」的冻结令，该冻结令随 E 修复作废。
- 缺陷登记：B-20 通用重绑的 God 审批流未实装（只登记，v1.1 处理）。

## D-037 备份粒度与落盘位置
- 日期：2026-09-23
- 状态：有效
- 决策：a) V 段每步前的回滚点只备份世界存档本体（最小文件集），全量冷拷仅在段首做一次；
  b) 所有备份落在非系统盘且非游戏/项目盘（当前定为 `F:\R4V-backups\`，记作 `<BACKUP_ROOT>`）；
  c) 每次备份前必须先报目标盘剩余空间，不足待拷体积 3 倍时停报。
- 理由：全量冷拷已达约 3.25 GB/次，其中约 3.04 GB 为**不变的非存档数据**——X-0.3a 实测
  该增量全属**另一个世界** `saves\save11`「Sanctum of Lemons」(29 城) 的
  `EmpireCraftModData.json`（6 件 2.87 GB）与 `Player-prev.log`（227 MB）；
  且三份备份曾全部堆在系统盘 C:，把可用空间压到 9.63 GB，风险波及 git / pytest / 游戏存盘。
- 影响面：X-0.3 的全量拷贝作为回滚点保留（已迁至 `F:\R4V-backups\`）；
  其后各步改用定向备份——**本测试世界（`saves\save10`「Mystery and Rain」）的最小文件集
  = map.meta + map.wbox + map_stats.s3db + EmpireCraftModData.json = 17,943,217 B**（X-0.3a.5）。

## D-038 冻结优先与 tick 指纹
- 日期：2026-09-23
- 状态：有效
- 决策：V 段任何验收动作一律「先冻结、后选目标」。冻结确认后记录该时刻 tick 为本轮
  **tick 指纹（TF）**，写入任务书与 SESSION-LOG；其后每一步执行前后各读一次
  get_world_state，tick 必须等于 TF，不等即全部基线作废并停报。
- 理由：R4-V 已三次因「先选目标后冻结」而基线作废（kingdom 17 灭 → 20 灭 → 22 改名 + 分裂）。
- 依据：D-033（tick 差值为唯一运行判据）。

## D-039 重绑目标改为「冻结后按目标城归属反查」
- 日期：2026-09-23
- 状态：有效
- 决策：不预先指定 kingdom_id。冻结后读 list_cities_ex，**城 id 6 归哪个 kingdom_id 就绑哪个**
  （记作 KB）；该值写入任务书后不得再变。
- 理由：世界自演化下任何预先指定的王国都会在执行前失效（D-035 即为实例）。

## D-040 tick 为帧计数、autosave 为实时驱动（移出运行判据集）
- 日期：2026-09-25
- 状态：有效
- 决策：**tick 为帧计数（约 59/s），不是世界时间，不得作运行判据；autosave 为实时驱动，
  移出判据集。**
- 依据：Z-3 全程观测（本地 `e64439435124fb24089c674341640b26aa622533` /
  镜像 `35c4513686f8fb1bf528835c87e5fc20465f8260`）：
  a) `pause`（2026-09-25 00:46:50.711，响应 `previous_paused:false / paused:true / tick=100380`）
     后全程 `paused` 字段为 true 且**世界确实冻结**（四个视觉检查点用户均答
     「单位不动 / 年份 813 逐字不变 / 按钮态=暂停」），而 tick 仍以 **≈59.1 tick/s** 增长
     （Z-3b 9×15 s INC=7123、Z-3c 12×60 s INC=39004、pause→结束 +50,636/857 s；STEPS 均为 0）；
  b) Z-1b 语义量快照 A（01:01:40.329 tick=152933）与 B（01:06:45.861 tick=170906）间隔 305.5 s，
     8 城 population / building_count / kingdom_id、王国数、kingdoms_ever_created、population_alive
     **逐字相同**，而 tick **+17,973（≥15,000）**；
  c) autosave 在冻结期间仍按 301 s 节拍出新槽（1790268384@00:46:24 → 1790268684@00:51:25 →
     1790268985@00:56:25）⇒ 实时驱动。
- **判据集（临时状态）**：
  `年份` / `population` / `building_count` / `王国数` —— **负路径已验证**（冻结期逐字不变），
    **正路径待验证**（运行态 5 分钟内有变化）；
  `单位移动` —— **仅人眼**；
  `paused` 字段 —— 本进程两次如实，**暂不恢复为判据**，B-18 状态不变。
- **待重算清单（只登记，不重算）**：ENV-1、C-3、B-21R、S-1.2、S-1.3、
  「三次王国基线作废」的归因。附注：**B-21R 可能源于 tick 判据误判，未验证**。
- 推翻条件：出现能独立证明 tick 与世界时间同步的证据（例如 tick 随存档持久、或与年份严格成正比）。

## D-041 世界运行判据（人眼 + API 双通道；tick / paused / autosave 不参与）
- 日期：2026-09-25
- 状态：有效
- 决策：**人眼通道** = 间隔 ≥60 s 两次读年份，1x 下有变化即判「运行」
  （依据：1x 下约 **50 s/年**、误差不超过 2 s；其他速度档另测）。
  **API 通道** = 300 s 窗口内 `population_alive` 或**任一城 `population`** 有变化即判「运行」；
  全部不变 = 判「冻结」。`building_count` **待验证**（取自 8723 `list_cities`，本轮无数据）。
  **`tick` / `paused` / `autosave` 不参与判定**（依据 D-040）。
- 依据：Z-3 负路径（冻结 15 分钟、语义量逐字不变）+ Z-7.2 正路径（`population_alive`
  1018→1045→1073、各城 `population` 大部变化、`population_lifetime` 948→948→1052、
  国名「清泉 国」→「清泉 直隶」）+ 用户实测年份流速（约 50 s/年，±2 s）。
- 证据位置：docs/SESSION-LOG/2026-09-22-R4-V.md【Z 段】Z-7.2 节（含三快照原始行）。
- 推翻条件：出现「300 s 窗口内 API 量全不变而世界确实在跑」（或反之）的实测反例。
