# ChatGPT / OpenAI GPT Image — engine guide

Self-contained guide for writing prompts for **OpenAI GPT Image** (ChatGPT's image generator and the Images API). Load this only when the user chose ChatGPT. Everything here is OpenAI-specific; ignore Gemini's rules.

## Models

| Model | Use it for |
|---|---|
| `gpt-image-2` | Default. Anything: photoreal, text, editing, reference images. What ChatGPT routes to. |
| `gpt-image-1.5` | Solid general model; use when you specifically need a **transparent background** (`background: "transparent"`). |
| `gpt-image-1-mini` | Cheap, fast drafts and high-volume batches. |
| DALL·E 3 | Legacy, retired May 2026. Only if the user explicitly opens the old DALL·E GPT. Don't ask it for legible text or exact object counts. |

GPT Image models are native and instruction-following: they parse full sentences, honour object counts and layout callouts, render legible text, and carry world knowledge. Write for that.

## The prompt formula

```
[Medium/style] of [subject with defining details] [action/pose], in [setting/background].
[Framing: shot type + camera angle + lens]. [Lighting]. [Colour palette]. [Mood]. For [intended use].
[Constraints last: exclusions, aspect ratio in words, "no watermark".]
```

For complex scenes, use labelled, line-broken segments (`Scene:`, `Subject:`, `Style:`, `Text (verbatim):`, `Constraints:`) — GPT Image follows them well and it cuts drift.

## Dimension vocabulary

- **Subject/action:** a full clause — subject doing something, with pose, gaze, object interaction. `a barista steaming milk while glancing toward the door`, not `barista, milk`.
- **Style/medium:** say `photorealistic` to trigger realism, then camera/physical words — `35mm film`, `50mm lens`, `shallow depth of field`, `film grain`, `visible skin pores`, `worn leather`. For stylised work lead with an explicit medium — `flat vector illustration`, `watercolour wash`, `cel-shaded`, `line art`. Concrete texture beats `8K`/`ultra-detailed`/`masterpiece` (ignored).
- **Composition/framing:** shot type (`close-up`/`wide`/`top-down`) + angle (`eye-level`/`low-angle`/`over-the-shoulder`) + lens (`24mm`/`85mm`) + placement (`centred with negative space on the left`, `rule-of-thirds, subject offset left`).
- **Lighting:** `soft diffuse light`, `golden-hour side light`, `high-contrast`, `rim lighting`, `backlit`, `natural window light`, `three-point studio setup`.
- **Colour:** name a palette — `cool blue-and-teal palette`, `muted pastels`, `warm autumnal tones`, `natural colour balance` — not raw hue lists.
- **Mood:** one or two words — `tense, cinematic` / `calm, airy` / `playful`.

## In-image text (a GPT Image strength)

1. Quote the exact string, in quotation marks or ALL CAPS.
2. Add `(EXACT, verbatim, no extra characters)` and `render once, clearly and legibly`.
3. Spell tricky/brand words letter-by-letter: `the word F-L-O-U-R`.
4. Typography as constraints: `bold sans-serif, high contrast, centred` — describe the style, don't expect exact font replication.
5. Keep display text short (1–3 words safest); list every label for infographics.
6. Set `quality: "high"` for small or dense text. GPT Image is best-in-class here, including non-Latin scripts.

## Editing and consistency

- **Iterate with the Responses API** (`previous_response_id`) for true multi-turn edits, or `images.edit` with the prior image plus a complete instruction. The plain one-shot Images endpoint has no memory.
- Every edit turn: state the one change **and** restate what stays. `increase warmth in the colour grading; keep the subject, composition, and background exactly the same.`
- **Localised edits:** `images.edit` accepts an alpha-channel **mask** (transparent = edit here), but mask precision is loose — explicit "change only X, keep the rest identical" text is more reliable.
- **Consistency across images:** pass the canonical reference to `images.edit` with `input_fidelity: "high"` (preserves faces/logos), and reuse the exact same description every time — paraphrasing causes drift. Up to 16 input images are combined as an undifferentiated set, so state which element comes from which image.
- Chaining many edits degrades quality; once you like a result, re-derive a full descriptive prompt of that state and regenerate fresh.

## Config / API reference

Endpoints: `images.generate`, `images.edit`, and the **Responses API** for conversational edits. Output is `b64_json` (decode it yourself). Every image carries **C2PA metadata + a SynthID watermark** (since May 2026).

| Param | Values | Notes |
|---|---|---|
| `model` | `gpt-image-2` · `gpt-image-1.5` · `gpt-image-1-mini` | Set explicitly. |
| `size` | `1024x1024`, `1536x1024`, `1024x1536`, `auto`; gpt-image-2 also arbitrary `WxH` (≤3840px edge, ratio ≤3:1, multiples of 16) | Also state the ratio in the prompt. |
| `quality` | `low` / `medium` / `high` / `auto` | Biggest lever for text fidelity. |
| `n` | 1–10 | Several variants in one call. |
| `background` | `transparent` / `opaque` / `auto` | Transparent needs `output_format` png/webp; solid on gpt-image-1/1.5 — verify current gpt-image-2 support before relying on it. |
| `output_format` | `png` / `jpeg` / `webp` | jpeg has no alpha. |
| `moderation` | `auto` / `low` | Two levels. |
| `input_fidelity` | `low` / `high` (edit) | `high` preserves faces/logos. |

```python
import base64
from openai import OpenAI
client = OpenAI()
r = client.images.generate(
    model="gpt-image-2",
    prompt=('Studio product photograph of a matte black aluminium water bottle, centred on a '
            'seamless light-grey background, soft three-point lighting, subtle reflection beneath. '
            'For an e-commerce listing. Square 1:1. No props, no text.'),
    size="1024x1024", quality="high", n=1,
)
open("bottle.png", "wb").write(base64.b64decode(r.data[0].b64_json))
```

## Anti-patterns

- Midjourney/SD flags (`--ar`, `::2`, `--v`) → ignored or rendered as text. Use the `size` param + prose.
- Keyword soup → write sentences.
- Negative prompts → no working field; phrase positively. Short single exclusions only, as a trailing clause.
- Vague adjectives (`beautiful`, `8K`, `epic`) → replace with material/light/colour/framing.
- Contradictions (two mediums, two framings) → pick one; split conflicting needs across a generate + an edit.

## Content policy

- **Real people:** public figures are broadly allowed since the 2025 loosening, but keep it non-sexual, non-violent, and non-misleading. Some figures are on an opt-out list and refuse regardless.
- **Copyrighted characters/logos:** direct reproduction often refuses; style homage ("Ghibli-style") is generally allowed.
- **Uploaded references:** you must have the rights to them.
- **Hard blocks:** sexual content involving minors, non-consensual intimate imagery, graphic gore, self-harm how-to, hate/harassment imagery.

## Worked examples (weak → strong)

**Photoreal:** `a nice photo of a person looking happy` →
> Photorealistic candid photograph of an elderly sailor on a fishing boat, weathered skin with visible pores, adjusting a rope net. Overcast harbour behind him. Shot on 35mm film, soft diffuse daylight, shallow depth of field, natural colour grading, subtle film grain. No text. Config: `size="1536x1024"`, `quality="high"`.

**Logo with text:** `cool logo for my bakery Field & Flour` →
> Original, non-infringing logo for a bakery. Wordmark text (EXACT, verbatim): "FIELD & FLOUR". A single wheat-stalk silhouette above the text, flat vector, minimal strokes, no gradients. Warm brown and cream palette. Typography: rounded serif, centred below the icon. Reads clearly at small sizes. No watermark, no extra text. Config: `background="transparent"`, `output_format="png"`, `quality="high"`.

**Infographic (dense text):** `infographic about coffee` →
> Clean infographic titled "HOW ESPRESSO IS MADE", portrait. Five numbered stages left to right, each a simple labelled icon connected by arrows. Labels (EXACT, verbatim): "1 GRIND", "2 DOSE", "3 TAMP", "4 BREW", "5 POUR". White background, single espresso-brown accent, legible sans-serif. Ensure exactly five stages, no extra text. Config: `size="1024x1536"`, `quality="high"`.

## Pre-send checklist

1. Natural sentences, ordered, intended use stated — not tags or flags?
2. One medium, one framing?
3. Every adjective renderable (material/light/colour/framing, not `8K`)?
4. Aspect ratio in words **and** `size` set?
5. In-image text quoted verbatim, `appears once`, typography specified, short; `quality:"high"` if dense?
6. Exclusions phrased positively, any "no X" placed last?
7. If editing: one change + full preserve-list this turn; Responses API or `images.edit` with `input_fidelity:"high"` for faces/logos?
8. Right model (gpt-image-2 default; gpt-image-1.5 for transparency; mini for drafts)?
9. Policy-safe (no misleading real-person depiction; nothing hard-blocked)?

Output the prompt + a Config block; optionally one Refine hint.
