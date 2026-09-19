# worldbox-ai-context

worldbox-ai 项目的**只读文档镜像**。

## 这是什么

把本地 `E:\work\worldbox-ai\docs\` 下的项目文档（状态、决策、记忆、架构稿、各轮 SESSION-LOG）
脱敏后镜像到公开仓库，供 AI 助手按 **commit SHA** 稳定引用。

## 什么不在这里

**源码与配置不在本仓库，且永远不进本仓库。** 具体排除：

- `.py` 源码（包括 `src\qqbot\`、`src\mcp-server\`、`src\wbai-plugin\`）
- `.dll` 二进制（游戏插件、桥接产物）
- `.sqlite3` 数据库（账本等）
- `.log` 日志
- `.toml` / `config.local.json` 等配置（含真实 token）
- `.venv-qqbot\` 虚拟环境
- `plugins\`、游戏目录、NapCat 安装目录下的任何内容

`.gitignore` 采用**白名单式**兜底：默认忽略一切，只放行 `docs/**/*.md`、`README.md`、`.gitignore`。
因此即使误放文件到仓库根，也不会被提交。

## 脱敏说明

镜像内容已脱敏，**不含**：

- 真实 QQ 号（全号）
- 真实 token 值（仅保留"token 长度"这类元信息，如 `token len=48`）
- 本机用户目录路径（如游戏存档目录、NapCat 安装目录整段替换为占位符）

保留的内容：

- 项目路径（`E:\work\worldbox-ai\...`）与游戏安装路径（`E:\game\worldbox`）——无敏感性，且引用时需要对齐
- 各组件/产物的**完整性校验值**（SHA256 / MD5）——这是校验指纹，不是机密
- 配置**键名**与端口号、以及"token 不入库/不入文档/不回显"这类设计约定

脱敏由每轮同步前的 grep 闸门把关（扫 QQ 号、token、长十六进制串、本机路径、password/secret 关键词），
**闸门未通过不推送**。

## 如何引用

固定格式（`<40位COMMIT_SHA>` 换成实际 SHA）：

```
https://raw.githubusercontent.com/<OWNER>/<REPO>/<40位COMMIT_SHA>/docs/<文件名>
```

GitHub raw 的结构是 `OWNER/REPO/REF/path/to/file`，REF 位置可以是分支名、tag 或 commit SHA。
**这里必须用 SHA**：分支名会随后续提交漂移，用分支名下次读到的就不是当时报告的那一份。

入口文件：`docs/INDEX.md`（每份文档的用途、字节数、最后更新轮次）。

## 同步约定

1. 每轮文档收束后同步一次，commit message 带轮次号（如 `R4-2c: sync docs`），并在汇报里给出新 SHA。
2. 旧 SHA 的链接依然有效——这正是用 SHA 而非分支名的意义。
3. 公开镜像只收脱敏后的 docs；源码永远留本地（agent 本来就直接读盘，不需要 GitHub）。