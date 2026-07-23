---
name: i-cant-draw
description: Translate a known image intent into an excellent OpenAI/ChatGPT image-generation prompt. Use this skill WHENEVER you are about to generate, request, or edit an image through ChatGPT, GPT-4o, gpt-image-1/1.5/2, or the OpenAI Images API — write an image prompt, help a user create/edit/restyle a picture, logo, poster, icon, infographic, product shot, or character. Trigger even when the user just says "make me an image of…", "draw…", "generate a picture of…", or "can you create a logo/poster/thumbnail". You already know WHAT image is wanted; this skill tells you HOW to phrase the prompt so the model renders it well.
---

# i-cant-draw

You know what image the user wants. Your job is to translate that intent into a prompt an OpenAI image model renders correctly on the first or second try. Write the prompt in full natural-language sentences, ordered, with constraints last. Then hand it to the user or call the image tool.

## What OpenAI image models want (and how they differ)

OpenAI's current models (gpt-image-2 is the ChatGPT default as of 2026; gpt-image-1/1.5 are legacy but prompt identically) are **autoregressive and instruction-following**, not diffusion tag-matchers like Midjourney/Stable Diffusion. This changes everything:

- **Natural language beats keywords.** The model parses grammar and relationships. Write a sentence, not a comma-salad of tags.
- **It follows instructions literally.** Compound requests, object counts, and layout callouts are honored — so state them.
- **Text rendering is strong.** It composes real letterforms because it understands the string semantically. You can ask for legible in-image text.
- **It has world knowledge.** Diagrams, labeled parts, and real-world facts work because generation is native to the LLM.
- **Under-described prompts don't fail — they get silently padded** with OpenAI's guesses. Specify anything you actually care about.

**Which model does what** (pick, don't agonize):

| Want | Use |
|---|---|
| Default / ChatGPT chat / anything | gpt-image-2 (what ChatGPT routes to) |
| Photorealism, editing, logos, reference images | gpt-image-1.5 / gpt-image-2 |
| Text-heavy, complex multi-object, infographics, multilingual, production | gpt-image-2 at `quality: high` |
| Cheap/fast drafts and batches | gpt-image-1-mini |
| Transparent background (PNG) | gpt-image-1.5 — **gpt-image-2 dropped transparency** |
| A distinct diffusion aesthetic only | DALL-E 3 (only if user explicitly opens the DALL-E GPT) |

Do NOT ask DALL-E 3 for legible text, exact object counts, or dense layouts — those are exactly its weak spots.

## The prompt formula

Order the prompt the way the model assembles the image: **background/scene → subject → action → composition/framing → style/medium → lighting → color → mood → constraints (last)**. State the **intended use** early (ad, UI mock, infographic, hero photo) — it sets the polish level.

Fill-in template:

```
[Medium/style] of [subject with defining details] [action/pose], in [setting/background].
[Framing: shot type + camera angle + lens]. [Lighting]. [Color palette]. [Mood]. For [intended use].
[Constraints last: exclusions, aspect ratio, "no watermark", any preserve notes.]
```

For **complex or multi-constraint** scenes, drop the paragraph and use labeled line-broken segments — it cuts drift:

```
Scene: …
Subject: …
Style: …
Lighting: …
Text (verbatim): "…"
Constraints: …
```

Worked example:

> Bad: `a nice photo of a person looking happy`
> Good: `Photorealistic candid photograph of an elderly sailor on a fishing boat, weathered skin texture, adjusting a net. Medium close-up, eye-level, shot on 35mm film with a 50mm lens and shallow depth of field. Soft overcast daylight. Muted natural color palette. Quiet, reflective mood. No glamorization, no text. 3:2 landscape.`

## Rules per dimension

### 1. Subject and action — describe a scene, not tags

Full clause: subject doing something, with pose, gaze, and object interaction.

Bad: `barista, milk, coffee shop`
Good: `a barista steaming milk while glancing toward the door, one hand on the pitcher`

### 2. Style/medium — name it once, concretely

Say `photorealistic` explicitly to trigger realism. For photoreal, use **camera/physical vocabulary**: `35mm film`, `50mm lens`, `shallow depth of field`, `film grain`, `visible skin pores`, `fabric wear`. For stylized, use **medium words** to suppress realism: `flat vector illustration`, `watercolor wash`, `cel-shaded`, `line art`.

Bad: `realistic detailed photo, 8k, ultra hd, masterpiece`
Good: `photorealistic close-up portrait, 85mm lens, shallow depth of field, visible pores and fine wrinkles, natural color grading`

**One medium per prompt.** Never mix `photorealistic` with `flat cartoon`, or `close-up` with `full-body wide shot` — the model blends them into mush.

### 3. Composition and framing — layered and explicit

Shot type (`close-up`/`wide`/`top-down`) + angle (`eye-level`/`low-angle`/`over-the-shoulder`) + lens (`24mm`/`85mm`) + placement (`centered with negative space on left`, `rule-of-thirds, subject offset left`, `logo top-right`). Word order = salience; the earliest subject reads as the focal point.

### 4. Lighting — name direction and quality

`soft diffuse lighting`, `golden-hour side light`, `high-contrast`, `rim lighting`, `backlit`, `natural window light`. Pair it with the mood it implies.

### 5. Color — name the palette, not raw hues

Bad: `red, blue, orange, teal`
Good: `cool blue-and-teal palette` / `muted pastel palette` / `warm autumnal tones` / `natural color balance`

### 6. Mood — one or two words

`tense, cinematic` / `calm, airy` / `playful`.

### 7. Aspect ratio — always state it in words

State the ratio in the prompt text **even when you also set the API `size` parameter, and even with a reference image supplied.** In the API: gpt-image-1/1.5 support only `1024x1024`, `1536x1024`, `1024x1536`, or `auto`; gpt-image-2 takes arbitrary `WxH` (multiples of 16, ratio ≤ 3:1, edge ≤ 3840px — treat anything above 2560x1440 as experimental).

Bad: `--ar 16:9` (flag syntax is ignored or rendered as literal text)
Good: `wide 16:9 landscape orientation` in prose, plus `size="1536x1024"` in the call.

### 8. Realism vs stylization — texture sells it

Concrete texture callouts (`worn leather`, `subtle film grain`, `everyday imperfections`) beat `8K ultra-detailed`. Fake camera specs (exact f-stop) are only loosely honored — use them for flavor, not precision.

## Rendering correct text in images

Text is a strength of the newer models — but only if you constrain it:

1. **Quote the exact string** verbatim, in quotation marks or ALL CAPS.
2. **Demand it literally:** add `(EXACT, verbatim, no extra characters)` and `render once, clearly and legibly`.
3. **Spell tricky/brand words letter-by-letter:** `the word F-L-O-U-R`.
4. **Typography as hard constraints:** `bold sans-serif, high contrast, centered, clean kerning` — not `nice font`.
5. **Bound repetition:** `ensure the text appears once` (prevents duplicated garbling).
6. **Bump `quality: high`** for small text, dense labels, or multi-font layouts.

Bad: `make a poster for my bakery called Fresh Loaf`
Good: `Poster for a bakery. Poster text (EXACT, verbatim): "FRESH LOAF". Typography: bold serif, warm brown, centered at top, appears once. No other text, no watermark. 2:3 portrait.`

**Keep display text short.** 1–3 words is safest; one short sentence works at `quality: high`. For many-label infographics, list every label explicitly by name — and for 10+ labels, generate panels separately and composite rather than forcing one shot. Non-Latin scripts (Japanese, Chinese, Hindi, etc.) are only reliable on gpt-image-2. Put long body copy as real text outside the image, never inside it.

## Editing and consistency

**Iterate in the same thread** (ChatGPT) or via `previous_response_id` (API) — a new chat loses the image context. Make **small, targeted** changes, not full rewrites.

**Phrase edits mechanically, not by vibe:**

Bad: `make it warmer` / `make it darker` / `make it better`
Good: `increase warmth in the color grading, add a slight orange/gold tint to the lighting` / `increase shadows and slightly reduce overall brightness`

**On every edit turn, state what changes AND restate what must stay** — nothing persists reliably, drift compounds:

> `Change only the sky to a sunset gradient. Keep the building, people, foreground, layout, and identity exactly as they are. Same 2:3 portrait.`

**For a consistent character/product across images:** keep one clean, front-facing canon reference, upload it every time (`use this character as reference, same face and outfit`), and reuse the *exact same* description and style phrasing every generation — paraphrasing causes visual drift. Chain images forward (each references the prior output); don't batch independent variants. Don't switch tools mid-project.

**Reference images / style transfer:** upload the reference and describe the transfer explicitly (`apply the color palette and brushwork of this reference to a new scene of X`) — there's no style-transfer flag. Note: the ChatGPT brush/selection tool and the API `mask` parameter are **loose guidance, not precise inpainting** on gpt-image-1/2 (confirmed limitation). For a localized edit, rely on explicit text (`only change the shirt to red, keep face, pose, and background pixel-identical`) rather than trusting the mask.

## ChatGPT chat vs gpt-image-1 API

- **ChatGPT** has conversational memory: follow-ups like `now make the jacket red` work. It may also silently rewrite/expand your prompt, so be explicit to reduce drift.
- **The API is stateless per call.** Every prompt must be fully self-contained — front-load all constraints (subject, composition, style, lighting, color, text-in-quotes, exclusions), because the model never asks clarifying questions. To iterate, either resend the whole prompt with the change folded in, or call `images.edit` with the prior image plus a complete instruction (`increase the otter's size to ~60% of frame, keep the vet, watercolor style, and lighting exactly as in the input`).
- API knobs worth setting: `quality` (`high` for text/portraits/dense), `size`, `input_fidelity: high` (edits, preserves faces/logos), `background: transparent` + `output_format: png` (gpt-image-1.5 only), `n` up to 10 for variants. GPT image models return `b64_json`, never a URL.

## Anti-patterns → do this instead

1. **Midjourney/SD flags** (`--ar`, `--v`, `::2`, `(word:1.5)`, seeds) → ignored or rendered as text. Use plain English + the real `size` param.
2. **Keyword soup** (`cyberpunk woman, neon, 8k, trending on artstation, masterpiece, octane render`) → reads as noise. Write a sentence.
3. **Negative prompts** → naming the excluded thing often renders it (`elephant without a fedora` → elephant in a fedora). Describe what you DO want (`an empty, quiet street`, not `no cars`). Short single exclusions (`no watermark`) are tolerated only as a trailing constraint, never the prompt's focus, never repeated.
4. **Vague adjectives** (`beautiful`, `nice`, `stunning`, `epic`) → no visual information. Replace with material, texture, light direction, color, framing, angle, medium.
5. **Contradictions in one prompt** → pick one medium/style/shot. Split conflicting needs across two images + an edit step.

## Content policy awareness

- **Real people / public figures**: generally allowed since the 2025 policy loosening — but keep it **non-sexual, non-violent, and non-misleading** (no false real-world scenarios / deepfakes). Some figures are on OpenAI's opt-out list and will refuse regardless of wording. Lower tiers may block uploading a photo of a real person for photo-to-art edits.
- **Copyrighted characters/styles**: mimicking a studio's or artist's *style* (e.g. "Ghibli-style") is generally permitted now; direct reproduction of specific characters/logos can still refuse, especially with commercial signals.
- **Hard blocks (never attempt):** sexual content involving minors, non-consensual intimate imagery, graphic gore, self-harm how-to, hate/harassment imagery. Not affected by any policy loosening.

## References

For a full library of ready-to-adapt prompt templates (billboard, ad tagline, movie poster, logo with brand name, dense infographic, product shot, UI mockup, character portrait, localization edit) and the reusable formula variants, read `references/prompt-cookbook.md`.

## Pre-send checklist

Before emitting the prompt, verify:

1. **Natural-language sentences**, not keyword tags or flags?
2. **Ordered** background → subject → action → framing → style → lighting → color → mood → constraints, with intended use stated?
3. **One medium/style only** — no contradictions?
4. **Concrete** — every vague adjective replaced with material/light/color/framing; realism cues are camera-based, not `8K`?
5. **Aspect ratio stated in words** (and `size` set if calling the API)?
6. **In-image text** quoted verbatim, `appears once`, typography specified, `quality: high` if dense — and short?
7. **Exclusions phrased positively**, kept to a brief trailing constraint (not negated mid-prompt)?
8. **If editing:** change + full preserve-list stated this turn, phrased mechanically?
9. **Policy-safe** — no misleading real-person depiction, no hard-blocked content?

If all yes, send the prompt (or call the image tool).
