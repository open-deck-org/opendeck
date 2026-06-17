# OpenDeck

> Build **animated, narrated HTML presentation decks** with any coding agent.

OpenDeck is an [Agent Skill](https://www.agensi.io/learn/agent-skills-open-standard): slides that reveal step-by-step as you click, hover tooltips, a thumbnail rail, fullscreen, print-to-PDF, and **AI voice narration** generated in-browser (ElevenLabs) that bakes into a fully offline file. Optionally package a deck as a portable `.deck` file for a compatible player app.

The skill follows the universal `SKILL.md` standard, so the **same skill folder works in Claude Code, OpenCode, Codex, Gemini CLI**, and other compatible agents.

```
opendeck/
├── .claude-plugin/          # Claude Code plugin + marketplace manifests
│   ├── marketplace.json
│   └── plugin.json
└── skills/
    └── opendeck/            # the portable skill (this is what every tool reads)
        ├── SKILL.md         # agent instructions
        ├── README.md        # human-facing kit guide
        └── assets/          # the shippable deck kit (engine, CSS, starter, schema)
```

---

## Install

### Claude Code

This repo is also a plugin marketplace, so installation is two commands:

```
/plugin marketplace add open-deck-org/opendeck
/plugin install opendeck@open-deck
```

(Or add it permanently to a project via `extraKnownMarketplaces` in `.claude/settings.json`.)

### OpenCode, Codex, Gemini CLI (and other `SKILL.md` agents)

These tools read skills from a local directory. Drop the `skills/opendeck/` folder into the right place:

| Agent        | Project-scoped                | User-scoped (global)                      |
| ------------ | ----------------------------- | ----------------------------------------- |
| OpenCode     | `.opencode/skills/opendeck/`  | `~/.config/opencode/skills/opendeck/`     |
| Codex        | `.codex/skills/opendeck/`     | `~/.agents/skills/opendeck/`              |
| Claude Code  | `.claude/skills/opendeck/`    | `~/.claude/skills/opendeck/`              |
| Gemini CLI   | (its configured skills dir)   | (its configured skills dir)               |

Fastest way to copy just the skill folder:

```bash
# requires Node (npx). Pulls only skills/opendeck/ — no git history.
npx degit open-deck-org/opendeck/skills/opendeck .opencode/skills/opendeck
```

Or clone and copy:

```bash
git clone https://github.com/open-deck-org/opendeck
cp -R opendeck/skills/opendeck ~/.config/opencode/skills/opendeck
```

---

## Use it

Once installed, ask your agent something like:

> *"Use the opendeck skill to build a narrated deck about our Q3 roadmap."*

The agent reads `SKILL.md` and scaffolds the deck from `assets/`. To add voice, open the deck and run `deckNarration.studio()` in the browser console (you supply your own ElevenLabs key — it never leaves your machine). See **[`skills/opendeck/README.md`](skills/opendeck/README.md)** for the full walkthrough and **[`skills/opendeck/SKILL.md`](skills/opendeck/SKILL.md)** for the complete build guide.

---

## License

[MIT](LICENSE) © Sinisha Djukic
