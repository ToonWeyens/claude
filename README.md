# Claude Code Commands

Custom slash commands for Claude Code.

## Install

```bash
git clone https://github.com/ToonWeyens/claude ~/Code/claude

mkdir -p ~/.claude/commands
mkdir -p ~/.claude/error-logs

ln -sf ~/Code/claude/CLAUDE.md     ~/.claude/CLAUDE.md
ln -sf ~/Code/claude/settings.json ~/.claude/settings.json
ln -sf ~/Code/claude/commands      ~/.claude/commands
ln -sf ~/Code/claude/error-logs    ~/.claude/error-logs
```

## Update

```bash
cd ~/Code/claude && git pull
```

## Commands

| Command | Description |
|---------|-------------|
| `/log-error` | Log and diagnose agentic coding errors, focused on improving user skill |
