Download: https://claude-seo.md/

## Writing agents

Claude Code reads the editable writing-agent profiles from `.claude/agents/`.
Codex requires generated TOML profiles under `.codex/agents/`.

When you edit, add, or remove a writing-agent profile, update
`.claude/agents/*.md` and then run:

```bash
python3 scripts/sync-agents.py
```

Do not edit `.codex/agents/*.toml` directly. Before finishing agent-profile
changes, check whether the generated Codex profiles are current:

```bash
python3 scripts/sync-agents.py --check
```
