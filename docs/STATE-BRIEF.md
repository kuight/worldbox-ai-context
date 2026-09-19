# STATE-BRIEF — 恢复指挥速览（供上下文压缩后整份粘贴）

> ★ 本文件供参谋侧（对话侧 AI）恢复用；执行侧不得据此行动，执行侧看 RUNNER-HANDOFF.md。
> 用途：Claude 上下文被压缩后，用户把本文件整份粘贴，即可恢复指挥所需最小信息。
> 更新时间：2026-09-16（R 轮）。

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