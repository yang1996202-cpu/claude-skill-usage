# Claude Code Skill Telemetry

统计你的 Claude Code Skills 真实使用频率。双数据源合并：解析历史 transcripts + Hook 实时埋点。

## 问题

装了 70 个 skills，不知道哪些在吃灰？Claude Code 没有内置的 skill 使用统计，transcripts 里的数据又不完整。

## 方案

```
历史数据 (projects/*.jsonl) ──┐
                              ├──→ skill-stats ──→ 使用报告
Hook 实时埋点 (jsonl) ────────┘
```

## 快速开始

### 1. 安装脚本

```bash
curl -fsSL https://raw.githubusercontent.com/yourname/claude-skill-telemetry/main/skill-stats \
  -o ~/bin/skill-stats && chmod +x ~/bin/skill-stats
```

### 2. 配置 Hook（实时埋点）

编辑 `~/.claude/settings.json`：

```json
{
  "hooks": {
    "PostToolUse": "if [ \"$CLAUDE_TOOL_NAME\" = \"Skill\" ]; then skill_name=$(echo \"$CLAUDE_TOOL_INPUT\" | python3 -c 'import sys,json; print(json.load(sys.stdin).get(\\\"skill\\\",\\\"unknown\\\"))'); echo '{\\\"timestamp\\\":\\\"'\"$(date -u +%Y-%m-%dT%H:%M:%SZ)\"'\\\",\\\"skill\\\":\\\"'\"$skill_name\"'\\\"}' >> ~/.claude/logs/skill-usage.jsonl; fi"
  }
}
```

**重启 Claude Code 生效。**

### 3. 查看统计

```bash
skill-stats              # 默认显示 Top 20
skill-stats --by-session # 按会话去重
skill-stats --detail web-access  # 单个 skill 详情
```

## 输出示例

```
#  │ Skill            │ 会话数 │ 消息数 │ Hook记录
── │ ──────────────── │ ─── │ ─── │ ──────
1  │ web-access       │ 12  │ 209 │ 5
2  │ checkpoint       │ 11  │ 188 │ 3
3  │ browse           │ 11  │ 841 │ 8
...

总计: 36 个 skill, 68 个会话, 2820 条消息, 16 次 hook 记录
```

## 注册为 Skill

复制 `SKILL.md` 到 `~/.claude/skills/skill-stats/`，之后可以：

- `/skill-stats` 直接触发
- 说"统计 skill"、"skill 排名"自动触发

## 数据源说明

| 数据源 | 路径 | 覆盖范围 | 局限 |
|--------|------|----------|------|
| Projects JSONL | `~/.claude/projects/*.jsonl` | 历史数据 | 只记录 attributionSkill，部分自动触发不写入 |
| Hook 日志 | `~/.claude/logs/skill-usage.jsonl` | 实时数据 | 需要重启 Claude Code 生效 |

## 原理

### 为什么 transcripts 数据不全？

Claude Code 的 transcripts 中：
- `attributionSkill` 只标记 assistant 消息归属
- 很多通过关键词自动触发的 skills 不写这个字段
- `tool_use` 中的 `Skill` 调用只占实际触发的一小部分

所以必须**双管齐下**：解析历史 + Hook 实时记录。

### 数据分两份正常吗？

正常。`~/.claude/transcripts/` 是旧版 OpenCode 数据，`~/.claude/projects/` 是当前 Claude Code 数据。只读 projects 即可。

## 扩展

- 按项目维度统计：projects 目录已按项目分组
- 按时间趋势：按天/周聚合
- 僵尸 skill 清理：对比 `~/.claude/skills/` 目录和统计数据

## License

MIT
