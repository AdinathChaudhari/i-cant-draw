<h1 align="center">🖍️ i-cant-draw</h1>
<p align="center"><strong>Great image prompts for ChatGPT and Gemini. No art skills needed.</strong></p>
<p align="center">
  <a href="LICENSE"><img src="https://img.shields.io/github/license/AdinathChaudhari/i-cant-draw?style=flat" alt="License"></a>
</p>

`i-cant-draw` is a skill that teaches an AI model to turn a rough image idea into a prompt that **ChatGPT (OpenAI GPT Image)** or **Google Gemini (Nano Banana)** actually renders well — a structured scene, real camera vocabulary, in-image text in quotes, and the right per-engine config. No keyword soup, no Midjourney flags.

It does **not** generate images. It generates the *prompt* you paste into the image tool.

```
You: "make me a logo for my coffee shop, Morning Brew, cosy and minimal"
      │
      ▼  i-cant-draw
Prompt: Original minimalist logo for a coffee shop. Wordmark text (EXACT, verbatim):
        "MORNING BREW". A single steaming-cup line-art icon above the text, flat vector,
        minimal strokes, no gradients. Warm brown and cream palette. Typography: rounded
        serif, centred below the icon. Plain white background, reads clearly at small
        sizes. No watermark, no extra text. Square 1:1.
        Config — ChatGPT: size="1024x1024", quality="high". Gemini: aspect_ratio="1:1".
      │
      ▼  paste into ChatGPT or Gemini
Result: a clean logo instead of a garbled blob.
```

## Supported engines

| Engine | Models it targets |
|---|---|
| **ChatGPT / OpenAI GPT Image** | `gpt-image-2` (default), `gpt-image-1.5`, `gpt-image-1-mini` |
| **Google Gemini / Nano Banana** | `gemini-3.1-flash-image` (Nano Banana 2), `gemini-3-pro-image` (Nano Banana Pro), `gemini-3.1-flash-lite-image` |

The prompt *body* is portable across both; the skill adds the per-engine config (aspect ratio, model choice, reference-image handling) that differs between them.

## Install (Claude Code plugin)

```bash
git clone https://github.com/AdinathChaudhari/i-cant-draw ./i-cant-draw
claude plugin marketplace add ./i-cant-draw
claude plugin install i-cant-draw@i-cant-draw
```

Then type `/i-cant-draw` and describe the image you want. Disable with `claude plugin disable i-cant-draw`. More in [INSTALL.md](./INSTALL.md).

## Use it anywhere else

`skills/i-cant-draw/SKILL.md` is plain markdown and model-agnostic. Paste it into a Custom GPT's instructions, a Gemini Gem, any assistant's system prompt, or a `.cursorrules` file. It teaches *how to prompt an image model*, so any model that writes the prompt can use it.

## What it changes

**Before** — habits carried over from Midjourney/Stable Diffusion:

> a poster for my coffee shop, cool and modern, coffee vibes, 8k, ultra detailed, trending, --ar 3:2, no ugly text

Keyword soup, a `--ar` flag both engines render as literal text, "no ugly text" that summons text, and vague adjectives the model has to guess at. You get a generic warm-tinted blob with a garbled word on it.

**After** — what the skill produces:

> Background: a warm exposed-brick wall in soft afternoon window light. Subject: a single steaming ceramic mug centred on a reclaimed-wood counter, photorealistic, shallow depth of field. Text: "MORNING BREW CO." across the top in a bold serif, rendered exactly once, verbatim, legible. Muted earth-tone palette, calm inviting mood. 3:2 landscape. No other text, no watermark.
>
> Config — ChatGPT: `size="1536x1024"`, `quality="high"`. Gemini: `aspect_ratio="3:2"`, `image_size="2K"`.

Scene ordered the way the model builds it, one medium, palette named, exact text quoted, aspect ratio in words *and* in the config, exclusions phrased positively and kept last.

## The core principles

Full skill in [SKILL.md](./skills/i-cant-draw/SKILL.md); a template + example library in [references/prompt-cookbook.md](./skills/i-cant-draw/references/prompt-cookbook.md). The short version — and both OpenAI and Google say the same things:

1. **Prose, not tags.** Full sentences describing a scene — never comma-separated keywords or `--flags`.
2. **Order it like the model builds it.** Background → subject → key details → style → constraints last.
3. **Say the exact words.** Concrete camera/lighting/palette terms ("35mm, shallow depth of field, muted teal palette"), not "beautiful" or "8k".
4. **Quote in-image text.** Exact string in quotes, spell tricky words out, name the font style — keep it short.
5. **Describe what you want, not what you don't.** Neither engine has a working negative-prompt field; one brief exclusion clause at the very end only.
6. **Mind the per-engine config.** ChatGPT takes pixel `size`; Gemini takes an `aspect_ratio` enum + `image_size` (and "4K" in the prompt is cosmetic there).

## Which engine when

- **Gemini** — character/subject consistency across a series, reference-heavy composition, conversational editing.
- **ChatGPT** — transparent-background assets, several variants from one call (`n`), a moderation-strictness lever.

Both watermark every output with SynthID by default, so neither produces an "undetectable" AI image.

## Tune it

Edit `skills/i-cant-draw/SKILL.md`, then re-invoke `/i-cant-draw`.

## Credits

Inspired by the direct, principle-first spirit of [i-have-adhd](https://github.com/ayghri/i-have-adhd) — pointed at image prompts instead of chat replies. Prompting guidance distilled from OpenAI's and Google's official image-generation docs (2026).

## License

MIT. Star ⭐ if it saved you a round of "why is there a garbled word on my poster?"
