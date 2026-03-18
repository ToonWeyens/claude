# Claude Code Commands

Custom slash commands for Claude Code.

## Install

```bash
git clone https://github.com/ToonWeyens/claude ~/Code/claude
mkdir -p ~/.claude/commands
ln -sf ~/Code/claude/commands/*.md ~/.claude/commands/
```

## Update

```bash
cd ~/Code/claude && git pull
```

## Commands

| Command | Description |
|---------|-------------|
| `/log-error` | Log and diagnose agentic coding errors, focused on improving user skill |
