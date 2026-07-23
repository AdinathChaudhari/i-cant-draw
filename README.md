<p align="center">
  <h1>🖍️ i-cant-draw</h1>
</p>
<p align="center">
  <strong align="center">Great image prompts. No art skills needed!</strong>
</p>
<p align="center">
  <a href="LICENSE"><img src="https://img.shields.io/github/license/AdinathChaudhari/i-cant-draw?style=flat" alt="License"></a>
</p>


## Install

```bash
git clone https://github.com/AdinathChaudhari/i-cant-draw ./i-cant-draw
claude plugin marketplace add ./i-cant-draw
claude plugin install i-cant-draw@i-cant-draw
```

In Claude Code: `/i-cant-draw`.

To disable: `claude plugin disable i-cant-draw` or use `/plugin disable i-cant-draw` from within CC.

More in [INSTALL.md](./INSTALL.md).

## What it does

A Claude Code skill that turns "make me an image of X" into a prompt gpt-image-2 actually renders right. Structured scene. Real camera words. Text in quotes. No keyword soup.

## What changes


<table>
<tr>
<td width="50%">

## Before

> a poster for my coffee shop, cool and modern, coffee vibes, 8k, ultra detailed, trending, --ar 3:2, no ugly text

Keyword soup + a Midjourney flag gpt-image-2 renders as literal text. "No ugly text" names text, so it draws some. Vague adjectives it has to guess at. You get a generic, warm-tinted blob with a garbled word on it.

</td>

<td width="50%">

## After

> Background: a warm exposed-brick wall in soft afternoon window light. Subject: a single steaming ceramic mug centered on a reclaimed-wood counter, photorealistic, shallow depth of field. Key details: the text "MORNING BREW CO." across the top in a bold serif, rendered exactly once, verbatim, legible. Muted earth-tone palette, calm inviting mood. 3:2 landscape. Constraints: no other text, no watermark.

Scene ordered the way the model assembles it, one medium, palette named, exact text quoted, aspect ratio in words, exclusions last.

</td>
</tr>
</table>


## The formula

Full skill in [SKILL.md](./skills/i-cant-draw/SKILL.md). The short version:

1. **Prose, not tags.** Full sentences describing a scene — never comma-separated keywords or `--flags`.
2. **Order it like the model builds it.** Background → subject → key details → style → constraints last.
3. **Say the exact words.** Concrete camera/lighting/palette terms ("35mm, shallow depth of field, muted teal palette"), not "beautiful" or "8k".
4. **Quote in-image text.** Exact string in quotes, spell tricky words out, name the font — keep it short.
5. **Describe what you want, not what you don't.** Negatives get rendered; one brief exclusion clause at the very end only.

## Tune it

Edit `skills/i-cant-draw/SKILL.md`. Re-invoke `/i-cant-draw`.

## The image-gen sibling of

[i-have-adhd](https://github.com/ayghri/i-have-adhd) — same direct, principle-first, no-fluff spirit, pointed at image prompts instead of chat replies.

## License

MIT.

Star ⭐ if it saved you one round of "why is there a garbled word on my poster?"
