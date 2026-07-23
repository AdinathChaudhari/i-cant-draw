---
name: i-cant-draw
description: Translate a known image intent into an excellent image-generation prompt for ChatGPT (OpenAI GPT Image) or Google Gemini (Nano Banana). Use this skill WHENEVER you are about to generate, request, or edit an image through ChatGPT, GPT-4o, gpt-image-1/1.5/2, the OpenAI Images API, the Gemini app, or the Gemini/Imagen API — or whenever you write an image prompt or help a user create, edit, or restyle a picture, logo, poster, icon, infographic, product shot, or character. Trigger even when the user just says "make me an image of…", "draw…", "generate a picture of…", or "can you create a logo/poster/thumbnail". You already know WHAT image is wanted; this skill tells you HOW to phrase the prompt so the model renders it well.
---

# i-cant-draw

You already know what image the user wants. Your job is to turn that intent into a prompt an image model renders correctly on the first or second try, and to phrase it for the right engine. Write in full natural-language sentences, ordered, with constraints last. Then hand the prompt to the user or call the image tool.

This skill covers the two engines most users reach for:

- **ChatGPT / OpenAI GPT Image** — `gpt-image-2` (default), `gpt-image-1.5`, `gpt-image-1-mini`. (DALL·E 3 was retired in May 2026; only use it if the user explicitly opens the legacy DALL·E GPT.)
- **Google Gemini / "Nano Banana"** — `gemini-3.1-flash-image` (Nano Banana 2, the Gemini app default), `gemini-3-pro-image` (Nano Banana Pro, best text and identity), `gemini-3.1-flash-lite-image` (fast/cheap). (Imagen is being shut down in 2026 — do not target it for new work.)

## Step 0 — pick the engine before you write

The prompt *body* is nearly identical for both engines; the *config* (aspect ratio, model, reference-image handling) differs. So:

1. If the user named a tool ("in ChatGPT", "with Gemini/Nano Banana"), write for that engine.
2. If they didn't, and it matters, ask once: **"ChatGPT or Gemini?"**
3. If you must produce something without asking, write the natural-language prompt (portable — it works in both) and add a short **"Config"** note for each engine (see the divergence table). Never block on this; a good portable prompt plus per-engine config is a complete answer.

## What both engines want (the shared foundation)

Both OpenAI and Google are emphatic on this, and it is the opposite of Midjourney/Stable Diffusion habits:

- **Describe the scene; don't list keywords.** Both vendors say this in almost the same words. The model parses grammar and relationships, so a narrative sentence beats a comma-salad of tags every time.
- **Be hyper-specific.** Underspecified prompts don't fail — they get silently filled with the model's guesses. Name anything you care about. "Ornate elven plate armour etched with silver leaf, high collar" beats "fantasy armour".
- **Order the prompt the way the image is built:** background/scene → subject → action → composition/framing → style/medium → lighting → colour → mood → constraints (last). State the **intended use** early (ad, UI mock, infographic, hero photo); it sets the polish level.
- **One medium, one framing.** Never mix "photorealistic" with "flat cartoon", or "close-up" with "full-body wide shot" — the model blends them into mush.
- **Phrase exclusions positively.** Neither engine's current models have a working negative-prompt field, and naming a thing to exclude tends to summon it ("an elephant without a fedora" → a fedora). Describe the wanted state: "an empty, deserted street", not "no cars". A short "no watermark, no extra text" is tolerated only as a trailing clause.
- **No Midjourney/SD syntax.** `--ar 16:9`, `--v`, `::2` weights, `(word:1.5)`, and seeds are ignored or rendered as literal text by both engines. Use plain English plus the real config field.
- **Iterate in small deltas.** One change per turn, and restate what must stay the same. Both engines drift if you cram many changes into one instruction.

## The prompt formula

```
[Medium/style] of [subject with defining details] [action/pose], in [setting/background].
[Framing: shot type + camera angle + lens]. [Lighting]. [Colour palette]. [Mood]. For [intended use].
[Constraints last: exclusions, aspect ratio in words, "no watermark", any preserve notes.]
```

For **complex or multi-constraint** scenes, drop the paragraph and use labelled, line-broken segments — both engines follow them well and it cuts drift:

```
Scene: …
Subject: …
Style: …
Lighting: …
Text (verbatim): "…"
Constraints: …
```

Worked example (portable across both engines):

> Bad: `a nice photo of a person looking happy`
> Good: `Photorealistic candid photograph of an elderly sailor on a fishing boat, weathered skin texture, adjusting a net. Medium close-up, eye-level, shot on 35mm film with a 50mm lens and shallow depth of field. Soft overcast daylight. Muted natural colour palette. Quiet, reflective mood. No text.`
>
> Config — ChatGPT: `size="1536x1024"`, `quality="high"`. Gemini: `aspect_ratio="3:2"`, `image_size="2K"`.

## Rules per dimension

### 1. Subject and action — a scene, not tags
Full clause: subject doing something, with pose, gaze, and object interaction.
Bad: `barista, milk, coffee shop`
Good: `a barista steaming milk while glancing toward the door, one hand on the pitcher`

### 2. Style / medium — name it once, concretely
Say `photorealistic` explicitly to trigger realism, then use **camera/physical vocabulary**: `35mm film`, `50mm lens`, `shallow depth of field`, `film grain`, `visible skin pores`, `worn leather`. For stylised work, lead with an explicit **medium**: `flat vector illustration`, `watercolour wash`, `cel-shaded`, `line art`. Concrete texture cues beat `8K`, `ultra-detailed`, `masterpiece` (SD-era tags both engines largely ignore).

### 3. Composition and framing — layered and explicit
Shot type (`close-up`/`wide`/`top-down`) + angle (`eye-level`/`low-angle`/`over-the-shoulder`) + lens (`24mm`/`85mm`) + placement (`centred with negative space on the left`, `rule-of-thirds, subject offset left`, `logo top-right`). Word order signals salience — the earliest subject reads as the focal point.

### 4. Lighting — name direction and quality
`soft diffuse light`, `golden-hour side light`, `high-contrast`, `rim lighting`, `backlit`, `natural window light`, `three-point studio setup`. Pair it with the mood it implies.

### 5. Colour — name the palette, not raw hues
Bad: `red, blue, orange, teal`
Good: `cool blue-and-teal palette` / `muted pastels` / `warm autumnal tones` / `natural colour balance`

### 6. Mood — one or two words
`tense, cinematic` / `calm, airy` / `playful`.

### 7. Aspect ratio — state it in words AND set the config
Always put the ratio in the prompt text, then set the real field. **The two engines take different inputs here:**
- **ChatGPT/OpenAI:** pixel `size` — `1024x1024`, `1536x1024`, `1024x1536`, or `auto` (gpt-image-1/1.5); gpt-image-2 accepts arbitrary `WxH` (multiples of 16, ratio ≤ 3:1, edge ≤ 3840px).
- **Gemini:** an aspect-ratio **enum** — `image_config.aspect_ratio` is one of `1:1, 3:2, 2:3, 3:4, 4:3, 4:5, 5:4, 9:16, 16:9, 21:9`, plus `image_size` of `"1K"`, `"2K"`, or `"4K"` (**uppercase K is required**).

Bad: `--ar 16:9` (flag syntax; rendered as literal text)
Good: `wide 16:9 landscape orientation` in prose, plus the config field for the chosen engine.

**Gemini gotcha:** writing "4K" or "HD" *inside the prompt* only nudges the *look* (sharpness cues) — it does **not** set pixel dimensions. Real resolution comes from `image_size`.

## Rendering correct text in images

Both engines render legible text — a genuine strength versus older diffusion models — but only if you constrain it:

1. **Quote the exact string** verbatim, in quotation marks or ALL CAPS.
2. **Demand it literally:** add `(EXACT, verbatim, no extra characters)` and `render once, clearly and legibly`.
3. **Spell tricky/brand words letter-by-letter:** `the word F-L-O-U-R`.
4. **Typography as hard constraints:** `bold sans-serif, high contrast, centred, clean kerning` — describe the style; don't name a specific font and expect exact replication.
5. **Keep display text short** — 1–3 words is safest. For many-label infographics, list every label explicitly.

Bad: `make a poster for my bakery called Fresh Loaf`
Good: `Poster for a bakery. Poster text (EXACT, verbatim): "FRESH LOAF". Typography: bold serif, warm brown, centred at top, appears once. No other text, no watermark. 2:3 portrait.`

**Per engine:**
- **ChatGPT (gpt-image-2):** best-in-class per most third-party tests, especially non-Latin scripts and dense multilingual layout. Use `quality: "high"` for small or dense text.
- **Gemini:** use **Nano Banana Pro (`gemini-3-pro-image`)** for anything text-heavy — it handles paragraph-scale text and infographics (keep total in-image text under ~400 words). A reliable Gemini trick: state the exact text content first, then describe the image around it.

## Editing and consistency

**Iterate in the model's own thread**, and make small, targeted changes. On every edit turn, state what changes **and** restate what must stay — nothing persists reliably, and drift compounds.

Phrase edits mechanically, not by vibe:
Bad: `make it warmer` / `make it better`
Good: `increase warmth in the colour grading, add a slight orange/gold tint to the lighting; keep the subject, composition, and background exactly the same`

**Per engine:**
- **ChatGPT:** use the **Responses API** (`previous_response_id`) for true multi-turn edits, or `images.edit` with the prior image plus a complete instruction. The plain one-shot Images endpoint does not remember earlier turns.
- **Gemini:** conversational editing is native and the *recommended* way to iterate. Use the **Interactions API** (`previous_interaction_id`) so you don't resend the image each turn. Append a preserve clause to every turn — Google's own examples literally say **"Do not change any other elements of the image."**

**Localised edits (inpainting):**
- **ChatGPT:** `images.edit` accepts an alpha-channel **mask** (transparent = edit here) — but treat mask precision as loose; explicit "change only X" text is more reliable.
- **Gemini:** no mask file needed — it does **semantic masking** from natural language: `Using the provided image, change only the blue sofa to a brown leather chesterfield. Keep everything else exactly the same, preserving the style, lighting, and composition.`

**Consistency across images (Gemini's headline strength):**
- Keep one clean canonical reference and reuse the **exact same** description every time — paraphrasing causes visual drift.
- **Gemini** splits reference images by **role**: object (up to ~10), character (up to ~4–5), style (up to ~3), 14 total on Nano Banana 2/Pro. Reference each by position ("the first image", "the second image") and say `Ensure the face and features remain completely unchanged`. For turnarounds, feed each generated angle back in as a reference for the next.
- **ChatGPT** takes up to 16 images to `images.edit` as an undifferentiated set to combine; use `input_fidelity: "high"` to preserve faces and logos.
- **Both:** quality degrades if you chain many edits (Gemini visibly after ~3–5 turns — each edit works on the previous output, like re-saving a JPEG). Once you like a result, re-derive a full descriptive prompt of that exact state and regenerate fresh rather than chaining forever.

## ChatGPT vs Gemini — the divergences that change your prompt

| Dimension | ChatGPT (OpenAI GPT Image) | Gemini (Nano Banana) |
|---|---|---|
| Current default / flagship | `gpt-image-2` | `gemini-3.1-flash-image` (app default); `gemini-3-pro-image` for text/identity |
| Aspect ratio | pixel `size` (`1536x1024`, `auto`, up to 3840px edge) | `image_config.aspect_ratio` **enum** + `image_size` `"1K"/"2K"/"4K"` |
| Editing API | Responses API (`previous_response_id`) or `images.edit` | Interactions API (`previous_interaction_id`); editing is native |
| Localised edit | mask file (loose) or "change only X" text | **semantic masking** from text — no mask file |
| Reference images | up to 16, undifferentiated array | up to 14, **split by role** (object/character/style) |
| Character consistency | good | **stronger** — a marketed first-class capability |
| In-image text | best-in-class, especially non-Latin scripts | excellent on **Nano Banana Pro**; text-first trick |
| Transparent background | `background: "transparent"` + PNG/WebP (solid on gpt-image-1/1.5; **verify for gpt-image-2 before relying on it**) | no documented transparent-background parameter |
| Multiple variants in one call | `n` up to 10 | 1 image per call (use Batch for volume) |
| Moderation control | `moderation: "auto" \| "low"` | no user-facing strictness dial — design around policy |
| Provenance watermark | C2PA metadata **and** SynthID (since May 2026) | SynthID, always on, non-optional |

**Quick "which engine" heuristic:** lean **Gemini** for character/subject consistency across a series, reference-heavy composition, and conversational editing; lean **ChatGPT** for transparent-background assets, several variants from one call (`n`), and a moderation-strictness lever. Both are SynthID-watermarked by default — never promise an "undetectable" AI image.

## Content-policy awareness

- **Real people:** **Gemini refuses identifiable real named people** (celebrities, politicians, anyone named) — tightened hard in the February 2026 safety upgrade; the only fix is to rephrase generically ("a woman in her 30s in business attire"). **ChatGPT** is more permissive for public figures since its 2025 loosening, but keep it non-sexual, non-violent, and non-misleading, and note that some figures are on an opt-out list and will refuse regardless.
- **Copyrighted characters / trademarked logos:** both try to block direct reproduction; expect refusals or generic substitutions. Style homage ("Ghibli-style") is generally allowed on both.
- **Uploaded references:** you must have the rights to any image you upload for editing or consistency.
- **Hard blocks (never attempt, on either engine):** sexual content involving minors, non-consensual intimate imagery, graphic gore, self-harm how-to, hate or harassment imagery.

## References

For a full library of ready-to-adapt templates, weak-vs-strong examples, a per-engine "same intent, two prompts" set, the vocabulary cheat-sheet, and side-by-side OpenAI and Gemini API quick-references, read `references/prompt-cookbook.md`.

## Pre-send checklist

Before emitting the prompt, verify:

1. **Engine chosen** (or a portable prompt plus per-engine config given)?
2. **Natural-language sentences**, ordered scene → subject → action → framing → style → lighting → colour → mood → constraints, with intended use stated — not keyword tags or flags?
3. **One medium/style, one framing** — no contradictions?
4. **Concrete** — every vague adjective replaced with material/light/colour/framing; realism cues are camera-based, not `8K`?
5. **Aspect ratio in words**, and the right config set for the engine (`size` for ChatGPT; `aspect_ratio` + `image_size` for Gemini — remember "4K" in the prompt is cosmetic)?
6. **In-image text** quoted verbatim, `appears once`, typography specified, short; `quality: "high"` (ChatGPT) or Nano Banana Pro (Gemini) if dense?
7. **Exclusions phrased positively**, kept to a brief trailing clause?
8. **If editing:** one change plus the full preserve-list this turn, phrased mechanically; Responses API (ChatGPT) or Interactions API (Gemini)?
9. **Consistency work:** same description reused verbatim; references passed by role (Gemini)?
10. **Policy-safe** — no identifiable real person on Gemini; no misleading real-person depiction on ChatGPT; no hard-blocked content?

If all yes, send the prompt (or call the image tool).
