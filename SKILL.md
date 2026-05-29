# skill-usage

## 描述

统计你的 Claude Code Skills 真实使用频率。双数据源合并：解析历史 transcripts + Hook 实时埋点。

## 触发方式

- `/skill-usage`
- 说"统计 skill"、"skill 排名"、"哪些 skill 在吃灰"

## 功能

- 自动探测 Claude Code 数据目录（支持 `~/.claude`、`~/.config/claude`、macOS `Application Support` 等常见路径）
- 合并历史 transcripts（`projects/*.jsonl`）和 Hook 实时日志（`logs/skill-usage.jsonl`）
- 按消息数或会话数排名，找出真正在用的 skill
- 支持 `--detail skill-name` 查看单个 skill 的使用详情

## 命名约定

- GitHub 仓库名：`claude-skill-usage`
- Claude slash skill 名：`skill-usage`
- 本地 CLI 名：`skill-stats`

不要把 CLI 文件名 `skill-stats` 当成 slash skill 名。Claude 实际识别的是你安装到 `~/.claude/skills/skill-usage/` 目录下的 skill。

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
