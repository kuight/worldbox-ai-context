# INDEX — worldbox-ai context 镜像索引

> 本文件是公开只读文档镜像的入口索引。新会话第一份读它。
> 镜像仓库：`worldbox-ai-context`（public）。源码、配置、数据库、日志、dll **不在此仓库**。
> 内容已脱敏：真实 QQ 号、token 值、本机用户目录路径均不存在（见 `README.md`）。
>
> **引用方式（务必带 commit SHA）**：
> `https://raw.githubusercontent.com/<OWNER>/worldbox-ai-context/<40位COMMIT_SHA>/docs/<文件名>`
> 用分支名会随后续提交漂移；用 SHA 可长期复现。

## 字段说明
- **字节数**：镜像内文件的实际大小（脱敏改动后）。
- **最后更新轮次**：取该文档内出现的最新轮次标记（可复现的客观规则）。
  ★ 注意：R3-W 收尾轮曾批量更新 MEMORY / DECISIONS / STATE-BRIEF / DESIGN-BRIEF 等，
  故个别文档的实际落笔轮次可能略晚于表中所示值。

## 主文档

| 文件 | 用途 | 字节数 | 最后更新轮次 |
|---|---|---|---|
| `AGENT-CONTRACT.md` | Agent 行为契约与铁律（脱敏纪律、运行模型记录约定；每轮开工前必读） | 7540 | K |
| `DECISIONS.md` | 决策与排除项（ADR 风格，只增不删；D-001~D-033） | 28012 | R4-V |
| `MEMORY.md` | 长期记忆，每轮累加的恢复依据（§1~§28，含组件哈希与实机结论） | 53393 | K |
| `STATE-BRIEF.md` | 参谋侧恢复指挥速览（环境栈/可行性/风险，整份粘贴用） | 3323 | R0 |
| `RUNNER-HANDOFF.md` | 执行侧换对话恢复用（当前任务书/动过的文件/闸门/缺陷登记） | 21982 | R4-V |
| `DESIGN-BRIEF.md` | 参谋侧设计要点与待查项 | 11493 | R3-V |
| `DESIGN-BRIEF-ADDENDUM.md` | 设计补充条目（C1-C6 / D1-D4） | 4555 | R3-X |
| `QQBOT-DESIGN.md` | QQ bot 产品设计稿（玩法/文案/平衡） | 8198 | R3-V |
| `QQBOT-V1-ARCH.md` | QQ bot v1 架构稿（八模块/文法/账本/回执/待批） | 13209 | R4-3 |

## SESSION-LOG（各轮原始记录）

| 文件 | 用途 | 字节数 | 最后更新轮次 |
|---|---|---|---|
| `SESSION-LOG/2026-09-14-A.md` | A 轮：环境勘查与组件盘点 | 3017 | A |
| `SESSION-LOG/2026-09-14-B.md` | B 轮：桥接安装与 token/鉴权实测 | 4987 | B |
| `SESSION-LOG/2026-09-14-C.md` | C 轮：MCP server venv 与自动发现缺陷 | 4050 | C |
| `SESSION-LOG/2026-09-14-E.md` | E 轮：存档/快照与 agents.json 边界 | 5760 | E |
| `SESSION-LOG/2026-09-14-G.md` | G 轮：screenshot 视觉通路 | 2702 | G |
| `SESSION-LOG/2026-09-14-I.md` | I 轮：cfg 编辑事故与整字节恢复 | 3113 | I |
| `SESSION-LOG/2026-09-14-J.md` | J 轮：含 token 备份删除与编码陷阱 | 4178 | J |
| `SESSION-LOG/2026-09-14-L.md` | L 轮：load_world 缺陷定性 | 3243 | L |
| `SESSION-LOG/2026-09-14-M.md` | M 轮：ILSpy 反编译研读 | 4794 | M |
| `SESSION-LOG/2026-09-14-N.md` | N 轮：power 打击类可行性 | 3258 | N |
| `SESSION-LOG/2026-09-14-O.md` | O 轮：插件骨架与 EmpireCraft 行为层 | 4647 | O |
| `SESSION-LOG/2026-09-14-P.md` | P 轮：WBAIBridge 自写插件与 8724 鉴权 | 12311 | P |
| `SESSION-LOG/2026-09-16-R0.md` | R0 轮：词汇/定性修正与文档幻觉清理 | 5577 | R0 |
| `SESSION-LOG/2026-09-16-R2.md` | R2 轮：WBAIBridge 0.2.0 与坐标精度结论 | 5302 | R2 |
| `SESSION-LOG/2026-09-16-R2-7to10.md` | R2-7~R2-10：延迟结算/Infinity 陷阱/暂停态投点 | 5129 | R2-10 |
| `SESSION-LOG/2026-09-17-R3.md` | R3~R3-4：list_cities_ex 可行性与编译修复 | 4079 | R3-4 |
| `SESSION-LOG/2026-09-17-R3-V.md` | R3-V：真机 16 城验证与偏差量化 | 3407 | R3-Y |
| `SESSION-LOG/2026-09-17-R4-0.md` | R4-0：NapCat 勘查四条结论（路径已脱敏） | 1371 | R4-0 |
| `SESSION-LOG/2026-09-19-R4-2.md` | R4-2 系列：离线骨架产出/用例演进(19→77)/SHA 链/偏差清单/字节数 | 6206 | R4-3 |
| `SESSION-LOG/2026-09-22-K.md` | K 轮：B-10 修复/D-031 接线/B-12 日志/协议表与真桥 fixture | 11132 | K |
| `SESSION-LOG/2026-09-22-R4-V.md` | R4-V 段：V-3 证据回灌（W-1 十二项原文/价目/备份/B-16/B-18/三份桥原文/逐城表） | 24805 | R4-V |

## 不在本镜像内的文档（有意排除）
以下原件存在于本地 `docs\`，但**未纳入**本镜像：
`BOOTSTRAP.md`、`FEASIBILITY-SELFHOSTED-COMMANDS.md`、`WORLDBOX-PROJECT-STATE.md`。
如需纳入，请在后续轮次的同步任务书中显式列出。

> **更正记录**：`AGENT-CONTRACT.md` 原不在 R4-G 的 G-1.2 复制清单内（任务书「后续约定」指定要往它写内容，
> 却漏放进清单）。经用户确认后已于本轮补入镜像，并**同样过完整脱敏闸门**（五条 grep + 宽模式盘符
> `[A-Za-z]:\` + 本机痕迹关键词 `Administrator|AppData|LocalLow|mkarpenko|Downloads`），结果无泄漏项。
> 未因它是「规矩文件」而豁免检查。

## 同步约定
1. 每轮文档收束后同步一次镜像，commit message 带轮次号（如 `R4-2c: sync docs`），报告新 SHA。
2. 旧 SHA 的链接依然有效——这正是用 SHA 而非分支名的意义。
3. 公开镜像只收脱敏后的 docs；源码永远留本地。