# skill-usage

## 描述

统计你的 Claude Code Skills 真实使用频率。双数据源合并：解析当前 Claude Code 会话 JSONL + Hook 实时埋点。

## 触发方式

- `/skill-usage`
- 说"统计 skill"、"skill 排名"、"哪些 skill 在吃灰"

## 功能

- 自动探测 Claude Code 数据目录（支持 `~/.claude`、`~/.config/claude`、macOS `Application Support` 等常见路径）
- 合并 `projects/*.jsonl` 和 Hook 实时日志（`logs/skill-usage.jsonl`）
- 按消息数或会话数排名，找出真正在用的 skill
- 支持 `--detail skill-name` 查看单个 skill 的使用详情

## 统计口径

- 真源 1：`~/.claude/projects/**/*.jsonl`
- 真源 2：`~/.claude/logs/skill-usage.jsonl`
- 非真源：`~/.claude/history.jsonl`
- 旧数据参考：`~/.claude/transcripts/*.jsonl`

不要临时写 Python 去扫 `history.jsonl` 代替正式统计，除非用户明确只想看 slash 命令历史。

## 命名约定

- GitHub 仓库名：`claude-skill-usage`
- Claude slash skill 名：`skill-usage`
- 本地 CLI 名：`skill-stats`

不要把 CLI 文件名 `skill-stats` 当成 slash skill 名。Claude 实际识别的是你安装到 `~/.claude/skills/skill-usage/` 目录下的 skill。

## 工作流

- 默认先跑 `skill-stats` 看按消息数的使用量，再跑 `skill-stats --by-session` 看覆盖度。
- 用户只问单个 skill 时，再补 `skill-stats --detail skill-name`。
- 回答前先说明统计口径：正式统计只认 `projects/**/*.jsonl` 和 Hook 日志；`history.jsonl` 只在用户明确要看 slash 历史时才使用。
- 不要临时写 Python 去扫 `history.jsonl` 代替正式统计。

## 输出格式

- 默认只输出 4 段：`统计口径`、`关键结论`、`Top 列表`、`异常/吃灰项`。
- `Top 列表` 默认最多给 Top 10；每条用一句人话解释，不要只贴数字。
- 不要直接粘贴 Bash 原始表格，除非用户明确要看原始输出。
- 不要把 `Thought`、`Bash`、`Successfully loaded skill`、截断提示、重复段落混进最终正文。
- 如果需要引用命令结果，只摘关键行并翻译成人话，不要把终端转储原样贴给用户。

## 依赖

- Python 3.8+
- 可选：配置 PostToolUse Hook 以获取实时数据（见 README）

## 安装

```bash
curl -fsSL https://raw.githubusercontent.com/yang1996202-cpu/claude-skill-usage/main/skill-stats \
  -o ~/bin/skill-stats && chmod +x ~/bin/skill-stats
```

## 用法

```bash
skill-stats                        # 默认显示 Top 20
skill-stats --by-session           # 按会话去重
skill-stats --detail web-access    # 单个 skill 详情
skill-stats --data-dir ~/.claude   # 手动指定数据目录
```
