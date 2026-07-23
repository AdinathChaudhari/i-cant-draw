# Install i-cant-draw

A Claude Code plugin. One skill inside.

## TL;DR

```bash
git clone https://github.com/AdinathChaudhari/i-cant-draw ./i-cant-draw
claude plugin marketplace add ./i-cant-draw
claude plugin install i-cant-draw@i-cant-draw
```

Open Claude Code, type `/i-cant-draw`.

To disable: `claude plugin disable i-cant-draw` (or `/plugin disable i-cant-draw` from within Claude Code). Re-enable later with `enable` instead of `disable`.

## Verify

```bash
claude plugin list
```

Look for `i-cant-draw  (enabled)`.

## Update

```bash
cd ./i-cant-draw && git pull
```

The marketplace re-reads the local checkout. Next Claude Code session picks up changes.

## Uninstall

```bash
claude plugin uninstall i-cant-draw
claude plugin marketplace remove i-cant-draw
```

## Always-on (optional)

To apply the prompting rules without typing `/i-cant-draw` first, add to `~/.claude/CLAUDE.md`:

```markdown
## Image generation

When generating or requesting any image via ChatGPT (OpenAI) or Gemini, follow the `i-cant-draw`
skill: natural-language description, subject-to-mood ordering, quoted literal text, the right
per-engine config, and no Midjourney flags.
```

## Use it anywhere (not just Claude Code)

`skills/i-cant-draw/SKILL.md` is plain markdown. Paste it into any assistant's system prompt,
a Custom GPT's instructions, a Gemini Gem, or a `.cursorrules` file. The skill is model-agnostic —
it teaches *how to prompt an image model* (ChatGPT or Gemini), so any model that writes the prompt
can use it.

## Troubleshooting

**`/i-cant-draw` not in autocomplete.** Restart Claude Code. The plugin index is read at startup.

**`claude plugin marketplace add` fails.** Point at the repo root, not at `.claude-plugin/`. The path must contain `.claude-plugin/marketplace.json`.

**Skill activates but prompts still feel weak.** Open a new session; old context may carry. If it still drifts, tighten the wording in `skills/i-cant-draw/SKILL.md`, then re-invoke.
