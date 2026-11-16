# Install wordpress-rescue

The skill is one folder: `wordpress-rescue`. Copy it into your agent's skills folder and the agent loads it whenever a task matches its description.

## Claude Code

```bash
mkdir -p ~/.claude/skills
cp -r wordpress-rescue ~/.claude/skills/
```

Restart Claude Code, or run `/skills` to confirm it is listed.

## Codex CLI

```bash
mkdir -p ~/.codex/skills
cp -r wordpress-rescue ~/.codex/skills/
```

## Cursor

```bash
mkdir -p .cursor/skills
cp -r wordpress-rescue .cursor/skills/
```

Cursor reads project skills from `.cursor/skills/` in the project root. For a global install, use `~/.cursor/skills/` instead.

## Claude.ai

Go to Settings, then Capabilities, then Skills, and upload the `wordpress-rescue` folder.

## Check it works

Ask your agent: "My WordPress site shows a white screen. Here is the debug.log:" and paste a log, or point it at `examples/sample-debug-log-fatal-plugin.log`.

The result should name the failing plugin, the file and line, the error class, the fix, and the rollback step, before it suggests changing anything. If the agent jumps to a fix without reading the log, point it at the first rule in SKILL.md.

## Updating

The free skill is updated on GitHub. Replace the folder with the new version. Your own site notes and backup routines live in your project, not in the skill, so an update will not overwrite them.
