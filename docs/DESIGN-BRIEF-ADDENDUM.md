# DESIGN-BRIEF 增补（2026-09-17，R3-V 后）
> 受众：参谋侧。本文件内容此前只存在于对话中。
> 与 DESIGN-BRIEF.md 并列使用；冲突时以本文件为准（时间更新）。

## A. 流程与治理（对话中拍板，未入 AGENT-CONTRACT）
A1 [用户拍板] 只读命令与"改游戏行为"命令必须分开做真机验证，不合并成一大轮。
   理由：目标指派风险比只读高一个量级，混在一起出问题归因会脏。
A2 [参谋·设计] 代码闸门必须贴 Read 的文件原文，不得凭记忆转述。
   事故：R3-1 汇报里出现 `d["id"] = city.data.id Leben;`，非法 C# 却报 0 错误编译通过
   → 审的文本与编译产物无关。事后 grep 确认源码干净，属转述失真。
A3 [参谋·设计] DLL 同一性只认 SHA256，禁止用字节数判据。
   事故：R3-3 与 R3-3b 均为 27,648 B 但内容不同（PE 512B 对齐填充掩盖了 IL 变化），
   agent 据此误报"产物与上轮相同"。
A4 [参谋·联网核实] "task tools haven't been used recently" 是固定节奏的系统注入，
   非模型生成、与模型档位无关（Claude Code issue 59213 记录 Opus 4.7 四小时触发约十次、
   四个 settings 位置均无法关闭；issue 62323 记录单场会话触发五百余次、
   设 todoFeatureEnabled=false 也无效）。Cherry Studio 提示文本逐字相同，推定同源。
   ★ 禁用 task 工具不能压制它——触发条件恰恰是"没用"这些工具，禁令适得其反。
   对策：任务书写"看到直接忽略、不为此中断步骤"，并定时换新会话（用户经验，有效）。
A5 [用户拍板] agent 卡住的主因是上下文过长，定时换新对话即可，不必归咎模型档位。
A6 [参谋·自省] grep 关键词须用文件原文字样，不得凭印象拟词。
   事故：R3-X 用"指派目标"核查第五节存档，实际原文为"目标指派"，0 命中被误判为缺节。

## B. 代码债（已知未修，本轮不动）
B1 HttpHealthServer 的命令白名单写在两处（if 排除判断 + 三元分派）。
   加第三条命令时漏改一处 → 404 说没有、或分派到错误命令。
   处置：加目标指派命令那一轮，一次收敛成字典或 switch。

## C. QQ bot 侧硬约束（技术实测推导，须写进实现）
C1 打击类回执必须两段式："已落地，结算中" → 5~10 秒后补发战果。
   伤害延迟结算，立即采样必然报 0（MEMORY §24-1）。
C2 播报前必须先判 max_warriors == 0，再看 ok_to_send。
   分母为 0 时浮点除零 = +Infinity，判定恒 true（§24-9，R3-V-7 实机确认）。
C3 warrior_slots / max_warriors 只能当"上一周期快照"，不得用于实时判断。
   实证：id=3 城 population 202→20、adults=11，warrior_slots 仍挂 112。
C4 不得用 is_alive 作为"城已毁"信号。isRekt() 看建筑全毁，非人口归零；
   is_alive == false 至今从未观测到，该边界未验证。
C5 调 list_cities_ex 后须与 8723 list_cities 的 count 对账，或世界加载后延迟首拉。
   世界加载中会静默返回不完整列表（首拉 1 城、复拉 6 城，代码无缺陷）。
C6 非暂停态投点前须就近重取坐标。移动目标从取坐标到落地之间世界会推进
   （R2-8d 打空的真因，R2-9b 已证 captain_x/y 字段本身准确）。

## D. 下一步路线（对话中调整，未入 DECISIONS）
D1 [用户拍板 2026-09-17] 插队顺序再调：QQ bot v1 消息链路 优先于 目标指派（D-017）。
   理由：纯 Python、不需用户在场开机。D-021 的插件优先级本身不变
   （list_cities_ex 已完成 → 目标指派 → 资源读写）。
D2 QQ bot v1 可用的桥接能力盘点：
   8723（worldbox-mcp 0.4.0，28 命令）——invoke_power / query_actors /
   list_cities / list_kingdoms / get_tile / population_alive /
   get_world_state / pause / resume / set_speed / list_speeds；
   8724（自写 WBAIBridge 0.3.0）——GET /health、POST /cmd：list_armies、list_cities_ex。
   ★ 按 D-004，QQ bot 直连 8723 的 /cmd，与 src\mcp-server 的一次性脚本通路互不干扰。
D3 待查（参谋自己联网，不占执行轮次）：NapCat / OneBot v11 正向 vs 反向 WebSocket。
   待执行侧查：用户本机那套 NapCat 现配置（哪种连接、是否仍可跑）。
   用户已确认本机此前部署过 NapCat。
D4 v1 仍以时间为货币（资源读写未落地），扣款内核在 v2 换真实资源，改动只在扣款一步。