# DESIGN-BRIEF — 参谋侧会话资产（2026-09-16）
> 受众：参谋侧（对话侧 AI）。执行侧看 RUNNER-HANDOFF.md。
> 性质：本文件内容此前只存在于用户与参谋的对话中，未进入 MEMORY / DECISIONS。
> 标签：[用户拍板] / [参谋·设计] / [参谋·推测] / [执行·实测]

## 一、角色与分工（2026-09-16 用户拍板）
- 参谋侧负责：QQ bot 设计、玩法设计、游戏平衡、经济构建；产出设计稿与任务书。
- 用户负责：拍板、GUI 操作。执行侧负责：取证、实装、落盘。
- 用户明示后续玩法设计「很大程度要靠参谋帮忙」，但不急，一步步来。

## 二、用户已拍板的玩法前提
1. [用户拍板] 不设领土限制：建造/增兵可落在任何人地盘（阴招算乐趣）。
   限制放在玩法层（价格、冷却），不放技术层（坐标校验）。
2. [用户拍板] bot 必须理解「我国 / 我城」概念，否则太不方便。
3. [用户拍板] 经济用游戏真实资源；GM 不做平衡；抢占好地区 → 发育更好（吃策略）。
4. [用户拍板] 初版保留用户否决权（终局才降格为普通玩家）。
5. [用户拍板] 微操要有：平时以布局为主，触发大战时需要微操。
6. [用户交回参谋决定] 一局时长 / 世界是否跨天延续；玩家身份与战败处理；PvP 或 PvE
   （天灾可由 mod 或 AI 扮演）。

## 三、微操的三档可行性（参谋更正了早前过严的说法）
- 指派目标 = 可行，但须自写插件走行为层。参考架构 = EmpireCraft（同版本同机正在运行）：
  ActorExtension 7 个 FrontLineMoveTarget 方法 + ConditionalWeakTable 自维护目标表
  + AssetManager.tasks_actor 注册 do_mod_actor_beh 行为节点参与 AI 决策。[执行·实测 P-4B]
- 强行阻断移动 = 已知失败路径，不要设计：ActorPatch.cs:149-152 的
  ShouldBlockMoveToByFrontLine 守卫被作者注释，moveTo prefix 注入点仍保留
  → 作者试过硬拦截并放弃。[执行·实测 P-4B-3]
- RTS 框选编队 = 无任何证据支持，排除（0.4.0 连移动命令都没有）。
- ★ 早前参谋说「只能造条件、不能指挥」是把版本限制说成玩法限制，已更正。

## 四、bot 的「我国/我城」实现方案 [参谋·设计]
> ★ 坐标部分已作废（R3-V）：城市目标改用 list_cities_ex 的 getTile().x/y 精确中心；
>   聚类取密度中心实测偏差 0~56 格且无规律，仅对移动目标保留近似口径。见 QQBOT-DESIGN 7.6。
- 正向解析（名字→目标）可行：QQ 侧维护绑定表（QQ号→kingdom_id，Python 自己的账本）；
  「我国」= 该 kingdom_id；「我城」= list_cities 按 kingdom_id 取；
  坐标 = query_actors 按 kingdom_id 拉单位后网格聚类取密度中心（N/O/P 轮靶点均此法）；
  首都另有捷径 = list_kingdoms 的 capital_id（也是扣款锚点）。
- 反向校验（坐标→归属）不可行：get_tile 无 city_id/zone/kingdom。
  故指令层只接受符号化目标（我城 / 我国边境 / 某国首都），不开放裸坐标输入；
  坐标由 bot 内部解析。摩擦力本身即劝退骚扰，不靠校验拦。
- ★ 已知误差：聚类中心是「单位分布中心」非「城市中心」，部队外出时会飘
  （O 轮地形阻断带实验因此判定数据不足）。落点是近似值 →
  商品说明必须写「区域打击，不是狙击」。

## 五、玩法内核草案 [参谋·设计，R3-10 修订]
★ 0.7f 出兵线不再作为玩法枢纽（D-020，2026-09-16）：四条杠杆全部实测报废——
  打平民延缓出兵（方向相反，静态证实分母随人口走）、打平民逼断补员
  （R2-8c 分母未塌缩，人口反升 641→658）、盖房钉兵（R2-8b 未成立）、
  打军队压分子（R2-9a 3 发减员 1，效率过低）。
  0.7f 仅作战况播报的只读指标，不作为可操作杠杆。
- living_house = 目前唯一验证过的增兵通路（O 轮 actor_count 2→5），用于催自己凑线。
- 墙类 + paint_tile 刷山 = 塑造战场、决定战争走向。
- 打击类定位降级为天灾（D-021）：战场变数 / PvE 天灾，非有效战斗手段；
  RTS 主轴为目标指派（D-017）；pause/set_speed 作为战斗节奏控制器。
- 部队不可微操（v1）→ 玩法是「做局」而非「微操」，也适配群里喊话的节奏。
★ spawn 作为增兵原语 = 未验证（Q 主任务书从未下发，"已验证"系幻觉，已落档）。

## 六、定价原则（反直觉，必须写进规则）[参谋·设计，依据 N 轮实测]
- N 轮数据：同坐标连续打击效果急剧递减（lightning 首发 -178，之后 earthquake /
  tornado / atomic_bomb 在同片废墟仅 -2~-4）。
- 结论：杀伤量取决于目标人口密度与打击顺序，与 power id 几乎无关 →
  按 power id 定价是错的（便宜 meteorite 砸满城是屠杀，核弹砸废墟是烟花）。
- 正确做法：按目标收费。每座城市一个独立「被打击冷却」，首发贵，
  冷却期内重复打同一城既贵又无效（规则明说，避免玩家以为被坑）。
  副作用：天然抑制针对单人的车轮战。

## 七、初版商品清单 [参谋·设计]
> ★ 本节已作废（R3-V）：现行清单见 QQBOT-DESIGN 第七节（D-022 按目标形态分工）。
>   本节保留作沿革记录，不得据此定价。
- 打击 5 样（均 N 轮 A 档）：meteorite（便宜试探）/ lightning（主力屠城）/
  earthquake、tornado（中档骚扰）/ atomic_bomb（大招，必过用户否决）。
- 建设 3 样：wall_iron、wall_ancient（A 档，60 秒+ 与跨会话持久）、living_house。
- 排除：angle_tower / flame_tower / monolith（O、P 两轮空地与密集区均 B 档，已停测）；
  clone_rain / equipment_rain（P-4C B 档）；spawn（未验证）。
- 待补测（便宜，下次有世界时顺手）：powers.txt 共七种墙
  （wall_ancient / wall_evil / wall_green / wall_iron / wall_light / wall_order / wall_wild），
  实测过 2 种且 2/2 命中；另有 ice_tower 从未测过。多几种墙 = 投屏更好看。

## 八、v1 / v2 两版路线 [参谋·设计]
- v1（今天就能跑）：因 8724 仍仅 /health、资源读写未落地，暂以「时间当货币」——
  每人按时间攒行动点。目的不是好玩，是跑通链路：群消息解析 → 符号化目标 →
  坐标解析 → POST /cmd → 回执 → 用户否决口子。
- v2：资源命令落地后把扣款内核换成真实资源（D-015），改动只在扣款那一步。
- 否决机制：便宜指令直接执行；atomic_bomb 与 paint_tile（不可逆/全图生效）
  进待批队列，用户在群里回一字放行或拒绝。并发无压力（≤5 人，桥接主线程串行 + 30 秒超时）。

## 九、插件优先级 [参谋·设计 + 用户同意]
list_armies（只读、零风险、微操与战况播报的共同前提，兼作 GameRefs/主线程模板的真机验证）
→ 资源读写（经济地基，写游戏状态）→ 目标指派（D-017 行为层）。
★ 前提：query_actors 只返回 name/asset_id/kingdom_id/x/y，asset_id 是物种（human）
不是兵种，无职业/城市/血量 → bot 现在「看不见军队」，故 list_armies 必须先行。

## 十、count 口径陷阱（参谋更正执行侧 R1-2）[执行·实测 + 参谋·推理]
- countUnits() = units.Count，**不判 isAlive**（MetaObject.cs:195-197）。
- countMelee() / countRange() 均带 isAlive() 过滤，且 countRange 还要求「有武器」。
- 故 countMelee + countRange ≤ countUnits，用前者算 0.7f 进度会**系统性低估**
  （bot 报「未到出兵线」而游戏已出兵；刚打完仗尸体未清时偏差最大，且不报错难查）。
- 要求：list_armies 必须同时返回 count_units，进度用它算；
  若 isOkToSendArmy() 为 public 则直接调游戏原生判定，不自己复现公式。

## 十一、外部事实（参谋联网核实，2026-09-16）
- PyPI worldbox-mcp 最新 = 0.6.0；不存在 0.7.0（连 GitHub tag 都没有）。
- 上游 compatibility.md：0.4.0 是唯一 ✅ 真机验证；0.5.0 / 0.6.0 均 🔵
  「nobody has run it against a game yet」；矩阵所有行只覆盖 0.51.2。
- 0.5.0 为 BREAKING：FactionPlayer 起不能调 invoke_power（返回 PERMISSION_DENIED）→
  升级会直接砸掉「群友作为派系玩家放神力」的玩法，这是留在 0.4.0 的新增强理由。
- 0.6.0 才给 invoke_power 加 radius / pulses / x2y2 拖拽（反证 0.4.0 无此参数）。
- ★ WorldBox 正式版已到 0.51.4（0.51.3 = 2026-05-15，0.51.4 = 2026-05-25），
  而整条栈（补丁版 NML + 0.4.0 bridge + 反射解析类结构）全部钉在 0.51.2 →
  **不要升级游戏**；若版本变动，全链路需重验。（此条尚未入 MEMORY）
- 上游 v0.1.0 发布说明自贴的 Assembly-CSharp SHA256 = 51d275f0… 与本机基线一致，
  为「官方原始构建」判定提供外部依据。

## 十二、Harmony 5 条警告 —— 已定性销账
- R1-4：EmpireCraft 源码三组 grep（loyalty+HarmonyPatch/AccessTools、WindowMetaGeneric、
  OnEnable+HarmonyPatch）全部 0 命中 → 非 EmpireCraft 所打。
- P 轮反射实测 bridge 的 8 个外部程序集不含 0Harmony → 非 bridge 所打。
- 故归属 NML。[参谋·推测] 机制是「日志去向改变而非新增故障」：
  HarmonyLib 的 AccessTools 警告走自身 FileLog 不入 Unity 日志；BepInEx core 自带
  同名 0Harmony.dll（HarmonyX）承接补丁调用后，警告被写进 LogOutput（前缀 [Warning: HarmonyX]）。
  支持证据：before-empirecraft 与 before-bepinex 两份基线 Harmony 命中数同为 15
  （装 EmpireCraft 后未新增任何行），且那 15 行是 IL Compile Error 异常堆栈（走 Unity 异常通道）。
- 结论：与桥接功能无关，观察项关闭。

## 十三、文档治理建议（部分已随 R1-5/R2-5 下发）
- 角色改按职能命名：参谋 / 执行（不用厂商或模型名，执行侧模型换过多次）。
- 每条结论加来源标签 [执行·实测] / [参谋·推测] / [上游文档]；
  **参谋的推测不得进入「已落地」段落**，升级须有执行侧实测原文。
  背景事故：Q-MEM 的 M1-5 是一句问句，被写 STATE-BRIEF 的模型当成既成事实
  写进 B3「已落地」→ 诞生「spawn 已验证」幻觉。
- 两份交接文档分受众：STATE-BRIEF（参谋）/ RUNNER-HANDOFF（执行），
  文件头互相声明「对方不得据此行动」。
- 索引里不记字节数（必然过时且无决策价值）；字节数只在 SESSION-LOG 记。
- 写入操作后必须 grep 核验（R0-5.5 的 Edit 曾静默失败，靠 grep 才抓到）。

## 十四、待办与悬置
- 参谋下一步：把本文件第五~八节写成 docs\QQBOT-DESIGN.md 的正式设计稿
  （填满那 7 条待决问题），交用户审。
- 待查（参谋自己联网，不占执行轮次）：NapCat / OneBot v11 正向 vs 反向 WebSocket 接入方式。
- 待执行侧查：用户本机那套 NapCat 现配置（哪种连接、是否仍可跑）。
- 已悬置不修：load_world 对本机存档格式的 0x7B 缺陷（定性不修，测试场景由用户手动读档）。
