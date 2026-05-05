# claude-skill-gdoc-sync

> **This skill has moved to the [`shaharsha/claude-skills`](https://github.com/shaharsha/claude-skills) monorepo.**

`gdoc-sync` is now part of a single repo containing all 9 of [@shaharsha](https://github.com/shaharsha)'s agent skills, with proper plugin marketplace support for Claude Code, Codex, Cursor, and other Agent-Skills-compatible harnesses.

## Install via Claude Code

```
/plugin marketplace add shaharsha/claude-skills
/plugin install shaharsha-skills@shaharsha-skills
```

(installs all 9 skills - or pick a subset with one of the category plugins; see the [monorepo README](https://github.com/shaharsha/claude-skills#install))

## Install just `gdoc-sync` manually

```bash
git clone https://github.com/shaharsha/claude-skills.git ~/shaharsha-skills
ln -s ~/shaharsha-skills/skills/gdoc-sync/ ~/.claude/skills/gdoc-sync
```

## Why the move

Single repo = one URL to share, plugin-marketplace native install, multi-agent manifests, accumulated stars on a single brand. See `obra/superpowers` and `anthropics/skills` for the dominant pattern. This repo is now archived (read-only).
