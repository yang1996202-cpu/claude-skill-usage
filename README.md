# Claude Code Skill Usage

> 追踪 Claude Code Skills 真实使用频率。双数据源合并：解析当前 Claude Code 会话 JSONL + Hook 实时埋点。

## 问题

装了几十个 skills，不知道哪些在吃灰？Claude Code 没有内置的 skill 使用统计，而会话 JSONL 里的数据又不完整 —— 很多自动触发的 skills 不会被记录到 `attributionSkill` 字段。

## 方案

```
历史数据 (projects/*.jsonl) ──┐
                              ├──→ skill-stats ──→ 使用报告
Hook 实时埋点 (jsonl) ────────┘
```

双管齐下，覆盖历史 + 实时，解决会话 JSONL 漏记问题。

## 真源约定

这套统计只认下面两个数据源：

- `~/.claude/projects/**/*.jsonl`：当前 Claude Code 的主会话数据，脚本会读取其中的 `attributionSkill` 和显式 `Skill` tool_use。
- `~/.claude/logs/skill-usage.jsonl`：你配置的 PostToolUse Hook 实时埋点。

下面这些不是当前脚本的真源：

- `~/.claude/history.jsonl`：更像 slash 命令/输入历史，只适合粗略看你手打过哪些命令，不适合当 skill 使用统计真源。
- `~/.claude/transcripts/*.jsonl`：通常属于 OpenCode/旧旁路会话数据，不是当前 Claude Code 的主会话真源，当前脚本默认不读它。

如果你看到别的 agent 临时写 Python 去扫 `history.jsonl`，那是一条旁路快诊，不代表这个仓库的正式统计口径。

## 官方文档

- Hooks 官方文档：<https://code.claude.com/docs/en/hooks>
- Settings 官方文档：<https://code.claude.com/docs/en/settings>

官方文档里已经明确说明：Hook 事件会通过 stdin 传 JSON，公共字段里包含 `transcript_path`，而当前设置与 Hook 配置都以 `~/.claude/settings.json` 为正式用户级入口。

## 快速开始

### 1. 安装

```bash
curl -fsSL https://raw.githubusercontent.com/yang1996202-cpu/claude-skill-usage/main/skill-stats \
  -o ~/bin/skill-stats && chmod +x ~/bin/skill-stats
```

### 2. 配置 Hook（实时埋点）

编辑 `~/.claude/settings.json`：

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Skill",
        "hooks": [
          {
            "type": "command",
            "command": "\"$HOME/bin/skill-stats\" --record-hook"
          }
        ]
      }
    ]
  }
}
```

如果你把 `skill-stats` 安装到了别的目录，把上面的 `$HOME/bin/skill-stats` 改成实际路径。

注意：Claude Code 当前要求的 Hook 结构是 `matcher + hooks[]`。旧的 `pattern/command` 写法会直接触发 settings error，整份配置会被跳过。

**重启 Claude Code 生效。**

### 3. 查看统计

```bash
skill-stats                        # 默认显示 Top 20
skill-stats --by-session           # 按会话去重
skill-stats --detail web-access    # 单个 skill 详情
skill-stats --top 10               # 只看前 10
skill-stats --data-dir ~/.claude   # 手动指定数据目录
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
数据目录: /Users/yang/.claude
```

## 数据目录探测

脚本会自动探测 Claude Code 数据目录，优先级如下：

| 优先级 | 来源 | 说明 |
|--------|------|------|
| 1 | `--data-dir` 参数 | 手动指定 |
| 2 | `CLAUDE_DATA_DIR` 环境变量 | `export CLAUDE_DATA_DIR=/path/to/.claude` |
| 3 | 自动扫描 | `~/.claude`、`~/.config/claude`、`~/Library/Application Support/Claude Code` 等 |

如果找不到数据，脚本会列出搜索过的路径并给出排查建议。

## 注册为 Skill（可选）

```bash
mkdir -p ~/.claude/skills/skill-usage
cp SKILL.md ~/.claude/skills/skill-usage/SKILL.md
```

之后可以：
- `/skill-usage` 直接触发
- 说"统计 skill"、"skill 排名"自动触发

## 保持本机与 GitHub 一致

如果你会在本机反复改这个仓库并来回 push/pull，推荐不要用 `cp`，改成软链，把本机安装点直接指向仓库真源：

```bash
mkdir -p ~/.claude/skills/skill-usage
ln -sfn /Users/yang/projects/claude-skill-usage/SKILL.md ~/.claude/skills/skill-usage/SKILL.md
ln -sfn /Users/yang/projects/claude-skill-usage/skill-stats ~/bin/skill-stats
```

这样你修改仓库里的 `SKILL.md` 或 `skill-stats` 后，本机会立即生效，不会再出现“GitHub 是一版，本机复制件又是另一版”。

## 数据源说明

| 数据源 | 路径 | 覆盖范围 | 局限 |
|--------|------|----------|------|
| Projects JSONL | `{data-dir}/projects/*.jsonl` | 历史数据 | 只记录 `attributionSkill`，自动触发 skills 可能漏记 |
| Hook 日志 | `{data-dir}/logs/skill-usage.jsonl` | 实时数据 | 需重启 Claude Code 生效 |

### 为什么会话 JSONL 数据不全？

- `attributionSkill` 只标记 assistant 消息归属
- 很多通过关键词自动触发的 skills 不写这个字段
- `tool_use` 中的 `Skill` 调用只占实际触发的一小部分

所以必须**双管齐下**：解析历史 + Hook 实时记录。

### 数据分两份正常吗？

正常。`~/.claude/projects/` 才是当前 Claude Code 的主会话数据；`~/.claude/transcripts/` 通常是 OpenCode/旧旁路残留数据。这个仓库只把 `projects` 和 Hook 日志当正式统计真源，不把 `transcripts` 混进来。

## License

MIT
