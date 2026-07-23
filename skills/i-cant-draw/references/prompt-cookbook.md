# Prompt Cookbook — i-cant-draw

Copy-pasteable library for writing image prompts for **ChatGPT (OpenAI GPT Image)** and **Google Gemini (Nano Banana)**. Companion to `SKILL.md`.

**How both engines work, in one paragraph:** OpenAI's GPT Image models (`gpt-image-2`, `gpt-image-1.5`) and Google's Gemini image models (`gemini-3.1-flash-image` / Nano Banana 2, `gemini-3-pro-image` / Nano Banana Pro) are *native multimodal* systems that reason over your words, not diffusion tag-matchers like Midjourney or Stable Diffusion. That is why both follow instructions, render real text, and edit conversationally. So for either: write ordered natural-language sentences (never keyword soup), state exclusions last, and describe what you *want* — not what you don't. The **prompt body is portable across both**; only the *config* (aspect ratio, model, references) differs.

---

## Part 1 — Weak vs. strong prompt pairs

Twelve pairs across common use cases. The prompt text works on both engines; the per-engine config is shown where it matters. Steal the structure, swap the nouns.

### 1. Photorealistic scene
**Weak:** `a nice photo of a person looking happy`
**Strong:**
> Photorealistic candid photograph of an elderly sailor on a fishing boat, weathered skin with visible pores and stubble, adjusting a rope net with both hands. Overcast harbour behind him. Shot on 35mm film, soft diffuse daylight, shallow depth of field, natural colour grading, subtle film grain. Wide 3:2 landscape. No text.

*Config* — ChatGPT: `size="1536x1024"`, `quality="high"`. Gemini: `aspect_ratio="3:2"`, `image_size="2K"`.
*Why it wins:* concrete texture + camera language ("visible pores", "35mm film", "shallow depth of field") triggers realism far harder than "nice" or "8K".

### 2. Illustration / stylised art
**Weak:** `cool fantasy forest, epic, trending, 8k, detailed`
**Strong:**
> Flat cel-shaded digital illustration of a moss-covered forest clearing at dusk. A single deer silhouette drinks from a still pond in the foreground. Warm bioluminescent mushrooms are the only light source. Muted teal-and-amber palette, soft painterly shadows, storybook mood. Vertical 2:3 portrait.

*Why it wins:* one explicit medium ("flat cel-shaded illustration") suppresses unwanted photorealism; a named palette and single focal subject stop the muddled blend "epic/trending/8k" produces.

### 3. Logo / graphic WITH text
**Weak:** `make a cool logo for my bakery called Field & Flour`
**Strong:**
> Original, non-infringing logo for a bakery. Wordmark text (EXACT, verbatim): "FIELD & FLOUR". A single wheat-stalk silhouette above the text, clean flat vector shapes, balanced negative space, minimal strokes, no gradients. Warm brown and cream palette. Typography: rounded serif, centred below the icon, generous padding. Plain white background, single centred logo. Reads clearly at small and large sizes. No watermark, no extra text.

*Config* — ChatGPT: for a true cut-out, `background="transparent"`, `output_format="png"`, `quality="high"`. Gemini: no transparent-background param — ask for a "plain white background" and cut it out afterward, or use Nano Banana Pro for the cleanest edges.
*Why it wins:* text quoted + marked verbatim, typography a hard constraint, and "reads clearly at small sizes / flat vector / minimal strokes" forces the simplicity a real logo needs.

### 4. Product shot
**Weak:** `product photo of a water bottle, professional`
**Strong:**
> Studio e-commerce product photograph of a matte black 750ml aluminium water bottle, centred on a seamless light-grey background. Soft three-point studio lighting with a gentle top highlight and a subtle reflection beneath the bottle. Sharp focus, true-to-life colour. No props, no hands, no text. Square 1:1.

*Why it wins:* names the exact studio setup and background catalogue shots actually use; excludes props/text that clutter listing images.

### 5. Character portrait
**Weak:** `portrait of a cyberpunk woman, badass, detailed, 4k`
**Strong:**
> Photorealistic close-up character portrait of a woman in her 30s with short silver hair and a faint neon-blue circuit tattoo along her jaw, wearing a matte black tactical jacket, looking directly at camera. Neon magenta and cyan rim lighting against a dark rain-slicked alley background. Shot as if on an 85mm portrait lens, shallow depth of field. Vertical 2:3.

*Why it wins:* every "badass" is replaced with a renderable detail (silver hair, circuit tattoo, rim lighting); the 85mm + shallow depth of field gives a real portrait look.

### 6. Infographic / diagram (text-heavy)
**Weak:** `infographic about coffee, informative, clean`
**Strong:**
> Clean infographic titled "HOW ESPRESSO IS MADE", portrait orientation. Five numbered stages left to right, each a simple labelled icon connected by arrows. Labels (EXACT, verbatim): "1 GRIND", "2 DOSE", "3 TAMP", "4 BREW", "5 POUR". White background, single espresso-brown accent colour used consistently, clear visual hierarchy, legible sans-serif labels. Ensure exactly five stages and no extra text.

*Config* — ChatGPT: `quality="high"`. Gemini: use **Nano Banana Pro (`gemini-3-pro-image`)** — it is the tier built for dense, legible text.
*Why it wins:* every label is listed explicitly and bounded ("exactly five"); dense multi-label work is the reliability edge — fewer labels per image means fewer garbles.

### 7. UI mockup
**Weak:** `app design for farmers market, modern UI`
**Strong:**
> Realistic mobile app UI mockup shown inside a phone frame. Top: a header with the market name and a search icon. Middle: a "TODAY'S SPECIALS" banner, then a scrollable vendor list with square photos and vendor names. Bottom: a tab bar with Home, Map, and Profile icons. White background, green-and-orange accent colours, clean sans-serif typography, generous spacing. Vertical 2:3.

*Why it wins:* lays out the screen region-by-region (top/middle/bottom) so the model places elements instead of inventing a generic "modern" blob.

### 8. Poster
**Weak:** `movie poster for a thriller`
**Strong:**
> Cinematic movie poster for a psychological thriller. Title text (EXACT, verbatim): "THE LONG DARK", set in bold condensed uppercase in the bottom third, with empty space beneath for credits. A lone figure stands at the end of a fog-filled street, backlit by a single sodium streetlight. High-contrast moody lighting, desaturated blue palette with one warm highlight. Vertical 2:3. Ensure the title appears once, legible. No other text, no watermark.

*Why it wins:* separates the display text (quoted, placed, bounded to once) from the pictorial description, and reserves credit space the way real key art does.

### 9. Sticker / icon (asset)
**Weak:** `an otter icon`
**Strong:**
> Kawaii-style sticker of a happy otter wearing a tiny explorer's hat, holding a smooth river pebble. Flat vector style, bold clean outlines, simple cel-shading, warm brown-and-cream palette, friendly expression. Single centred subject, plain white background, no text.

*Config* — ChatGPT: `background="transparent"`, `output_format="png"`. Gemini: no alpha param — request a plain white background and key it out afterward.
*Why it wins:* "single centred subject / flat vector / bold outlines / plain background" is exactly Google's documented sticker pattern and keeps it asset-clean.

### 10. Multi-image fusion (Gemini strength)
**Weak:** `put my product in a nice scene`
**Strong (Gemini):**
> Create a professional e-commerce fashion photo. Take the blue floral dress from the first image and let the woman from the second image wear it. Generate a realistic full-body shot of the woman wearing the dress, with the lighting and shadows adjusted to match an outdoor golden-hour environment. Keep her face and the dress pattern unchanged. Vertical 4:5.

*Config* — Gemini: pass the two images as input parts (order matters — "first image", "second image"). ChatGPT: pass both to `images.edit` and describe the combination; there is no role split, so be explicit about which element comes from which image.
*Why it wins:* names each source element by image position and states what must stay unchanged — Gemini's role-based reference system rewards this.

### 11. Multilingual / non-Latin text
**Weak:** `a Japanese festival poster with some Japanese text`
**Strong:**
> Festival poster in a woodblock-print style. Headline text (EXACT, verbatim, Japanese): "夏祭り". Below it in smaller type (EXACT): "2026". Lanterns and fireworks over a river at night, warm red-and-gold palette. Headline centred at top, large and legible. Vertical 2:3. No other text.

*Config* — ChatGPT (`gpt-image-2`) is the safest default for non-Latin scripts. Gemini: use Nano Banana Pro and state the text first, then the scene.
*Why it wins:* quotes the exact glyphs and keeps the string short — both engines garble long non-Latin passages.

### 12. Iterative edit / localisation (constraint-heavy)
**Weak:** `make it in Spanish, keep it looking the same-ish`
**Strong:**
> Update this image so all visible text is in Spanish, translated accurately and verbatim. Change only the text. Keep everything else exactly the same: layout, typography style, spacing, text hierarchy, colour palette, logo position, and background imagery. Do not reflow unless a translated string genuinely does not fit.

*Config* — Gemini: send as an Interactions turn with `previous_interaction_id`; Google's own phrasing is "Do not change any other elements of the image." ChatGPT: use the Responses API or `images.edit` with `input_fidelity="high"`.
*Why it wins:* an edit succeeds or drifts based on the *preserve list* — spell it out, every turn (see Part 5).

---

## Part 2 — Reusable template formulas

Fill the brackets. Keep the ordering; put exclusions last. Bodies are portable; set the config per engine.

### Template A — Photoreal scene
```
Photorealistic [shot type] of [subject + defining physical details]
[doing action], in [setting/background].
[Lighting: e.g. soft overcast daylight / golden-hour side light].
Shot on [lens, e.g. 35mm film / 85mm portrait lens], [depth-of-field cue].
[Colour grading]. [Texture callouts: pores, fabric wear, grain].
[Aspect ratio in words]. [Exclusions: no text, no watermark].
```

### Template B — Illustration / stylised
```
[Explicit medium: flat vector / watercolour / cel-shaded / line art]
illustration of [subject + action], in [setting].
[Named colour palette]. [Light source]. [Mood].
[Composition + focal subject]. [Aspect ratio]. [Exclusions].
```

### Template C — Graphic / logo WITH text
```
[Graphic type] for [entity/purpose].
[Text role] text (EXACT, verbatim): "[EXACT STRING]".
[Icon/imagery description]. [Style: flat vector / minimal strokes].
[Named palette]. Typography: [font style, weight, case], [placement],
[size relationship to icon]. [Background treatment].
Ensure the text appears once and is perfectly legible.
No watermark, no extra text.
```

### Template D — Product shot
```
Studio product photograph of [product + material + size], centred on
[seamless background colour]. [Lighting setup, e.g. soft three-point with top highlight].
[Reflection/shadow treatment]. Sharp focus, true-to-life colour.
For [use, e.g. an e-commerce listing]. [Aspect ratio].
No props, no hands, no text.
```

### Template E — Multi-image fusion (Gemini)
```
Create [final scene type]. Take the [element] from the first image and
[combine/place] it with the [element] from the second image.
The final image should be [description of the final scene].
Ensure the [face/logo/pattern] from [image N] remains completely unchanged.
[Aspect ratio].
```

### Template F — Iterative / localised edit (both engines)
```
[Single change: "change only X to Y" / "translate all text to Z"].
Keep everything else exactly the same — [explicit list: layout, colours,
camera angle, background, identity/face, pose, logo position, typography].
[Restate this whole preserve-list on EVERY follow-up turn.]
```

---

## Part 3 — Vocabulary cheat-sheet

Words that steer both engines. Prefer these over `8K` / `masterpiece` / `trending on artstation` (SD-era tags, largely ignored by both).

| Dimension | Words that steer the model |
|---|---|
| **Composition / framing** | close-up, medium shot, wide shot, extreme wide, top-down/overhead, over-the-shoulder, rule-of-thirds, centred, negative space on [side], symmetrical, foreground/midground/background, tight crop |
| **Camera angle** | eye-level, low-angle, high-angle, worm's-eye, bird's-eye, Dutch tilt, straight-on, three-quarter view, profile |
| **Camera / lens** | 24mm wide lens, 35mm film, 50mm, 85mm portrait lens, macro, shallow depth of field, deep focus, bokeh background, motion blur, long exposure, film grain, lens flare |
| **Lighting** | soft diffuse light, golden hour, blue hour, high-contrast, harsh overhead, rim lighting, backlit, side light, natural window light, three-point studio, neon glow, chiaroscuro, overcast |
| **Art style / medium** | photorealistic, 35mm film photograph, flat vector illustration, watercolour, gouache, cel-shaded, line art, ink drawing, 3D render, isometric, pixel art, oil painting, charcoal sketch, woodblock print, claymation |
| **Colour palette** | warm autumnal palette, cool blue-and-teal tones, muted pastels, high-saturation neon, monochrome black-and-white, earth tones, desaturated, complementary orange-and-blue, natural colour balance |
| **Mood** | serene/calm, tense, cinematic, whimsical, melancholic, energetic, ominous, cosy, minimalist, nostalgic, dramatic, playful |

**Realism dial:** to push realism, add texture cues (visible pores, skin wrinkles, fabric wear, worn metal, everyday imperfections). To suppress it, lead with an explicit illustration medium (flat vector, watercolour wash, cel-shaded).

---

## Part 4 — API quick reference (side by side)

Both APIs are stateless per call: the chat app's implicit memory does not exist, so every prompt must be fully self-contained.

### ChatGPT / OpenAI GPT Image

Endpoints: `images.generate` (text→image), `images.edit` (image[+mask]→image), and the **Responses API** for conversational multi-turn edits. Returns `b64_json` (decode it yourself).

| Param | Values | Notes |
|---|---|---|
| `model` | `gpt-image-2` · `gpt-image-1.5` · `gpt-image-1-mini` | Set explicitly; otherwise older defaults apply. |
| `prompt` | string | Fully self-contained — no memory between calls. |
| `n` | 1–10 | Several variants in one call. |
| `size` | `1024x1024`, `1536x1024`, `1024x1536`, `auto`; gpt-image-2 also arbitrary `WxH` (≤3840px edge, ratio ≤3:1, multiples of 16) | Also state the ratio in the prompt. |
| `quality` | `low` / `medium` / `high` / `auto` | Biggest lever for text fidelity. |
| `background` | `transparent` / `opaque` / `auto` | Transparent needs `output_format` png/webp; solid on gpt-image-1/1.5, verify current gpt-image-2 support. |
| `output_format` | `png` / `jpeg` / `webp` | jpeg has no alpha. |
| `moderation` | `auto` / `low` | Only two levels. |
| `input_fidelity` | `low` / `high` (edit) | `high` preserves faces/logos/fine detail. |

```python
import base64
from openai import OpenAI
client = OpenAI()

r = client.images.generate(
    model="gpt-image-2",
    prompt=('Studio product photograph of a matte black aluminium water bottle, '
            'centred on a seamless light-grey background, soft three-point lighting, '
            'subtle reflection beneath. For an e-commerce listing. Square 1:1. No props, no text.'),
    size="1024x1024",
    quality="high",
    n=1,
)
open("bottle.png", "wb").write(base64.b64decode(r.data[0].b64_json))
```

### Google Gemini / Nano Banana

Two surfaces: `generateContent` (legacy but fully supported) and the newer **Interactions API** (recommended, threaded via `previous_interaction_id`). Imagen's `predict` endpoint is being shut down in 2026 — don't build on it. Every output carries a non-optional SynthID watermark.

| Config | Values | Notes |
|---|---|---|
| `model` | `gemini-3.1-flash-image` (Nano Banana 2, default) · `gemini-3-pro-image` (Pro, best text/identity) · `gemini-3.1-flash-lite-image` (cheap) | Pick Pro for dense text or identity work. |
| `response_modalities` | `["IMAGE"]` | Required for image output on `generateContent`. |
| `image_config.aspect_ratio` | `1:1, 3:2, 2:3, 3:4, 4:3, 4:5, 5:4, 9:16, 16:9, 21:9` | An enum, not pixels. Omit → inferred from the input image. |
| `image_config.image_size` | `"1K"`, `"2K"`, `"4K"` | **Uppercase K required.** Sets real resolution — "4K" in the prompt text does not. |
| reference images | up to 14 total, split by role | object (~10) / character (~4–5) / style (~3) on Nano Banana 2/Pro. |
| negative prompt | *not supported* | Phrase exclusions positively instead. |

```python
from google import genai
from google.genai import types
client = genai.Client()

r = client.models.generate_content(
    model="gemini-3.1-flash-image",
    contents=("Studio product photograph of a matte black aluminium water bottle, "
              "centred on a seamless light-grey background, soft three-point lighting, "
              "subtle reflection beneath. For an e-commerce listing. Square. No props, no text."),
    config=types.GenerateContentConfig(
        response_modalities=["IMAGE"],
        image_config=types.ImageConfig(aspect_ratio="1:1", image_size="2K"),
    ),
)
for part in r.parts:
    if part.inline_data:
        part.as_image().save("bottle.png")
```

Multi-turn edit (Interactions API):
```python
first = client.interactions.create(model="gemini-3.1-flash-image", input="Create a vibrant infographic of the water cycle.")
second = client.interactions.create(
    model="gemini-3.1-flash-image",
    input="Update this infographic to be in Spanish. Do not change any other elements of the image.",
    previous_interaction_id=first.id,
)
```

---

## Part 5 — Iterative-editing phrasebook

**Golden rule:** stay in one thread, change one thing per turn, and restate the preserve-list every time. Nothing persists reliably — drift compounds, and on Gemini image quality visibly degrades after ~3–5 chained edits.

**Phrase edits mechanically, not by vibe:**

| Instead of (vague) | Say (mechanical) |
|---|---|
| make it warmer | increase warmth in the colour grading, add a slight orange/gold tint to the lighting |
| make it darker | increase the shadows and slightly reduce overall brightness |
| make it pop | increase saturation and add stronger rim lighting on the subject |
| make it cleaner | remove background clutter, increase negative space, simplify to one accent colour |
| better composition | recompose to rule-of-thirds with the subject offset left, more headroom |

**Reliable edit instructions (both engines):**
- Reframe: `zoom out to show the full body` · `crop tighter to a head-and-shoulders close-up` · `extend the scene to the left`
- Add/remove: `add a small dog sitting in the background; keep everything else the same` · `remove the coffee cup on the table, fill with matching wood grain`
- Recolour one thing: `change only the jacket to deep red; keep the face, pose, and background exactly the same`
- Time/weather: `same scene at night, neon signs lit; keep the layout and subject identical`

**Preserve-list to append (pick what matters):**
> Keep everything else exactly the same: layout, camera angle, the subject's face and identity, pose, outfit, background, colour palette, logo position, typography.

**Consistency across a series (no seed exists on either engine):**
1. Make one clean, front-facing **canonical reference** image.
2. Reuse the **exact same** character description and style phrase — never paraphrase (synonym drift = visual drift).
3. **Gemini:** upload the reference in the **character role** and say "keep the same face and outfit"; for turnarounds feed each generated angle back in. **ChatGPT:** pass the reference to `images.edit` with `input_fidelity="high"`.
4. Chain forward: each new image references the previous output, changing only scene/action. Don't batch independent variants.
5. Don't switch engines mid-project — the same prompt renders differently on each.

---

## Part 6 — Anti-patterns (delete these on sight, both engines)

- **Midjourney/SD flags:** `--ar 16:9`, `--v 6`, `--stylize`, `::2`, `(word:1.5)`, `--seed`. Ignored or rendered as literal text. Use plain English + the config field (`size` for ChatGPT, `aspect_ratio` for Gemini).
- **Keyword soup:** `cyberpunk woman, neon, 8k, ultra detailed, trending, octane render`. Both vendors explicitly say: describe the scene, don't list keywords. Write a sentence.
- **Negative prompts:** neither engine's current models have a working negative-prompt field, and naming the excluded thing tends to render it. Describe the positive state (`a bare-headed elephant`, `an empty street`). Short single exclusions are fine only as a trailing clause.
- **"4K"/"HD" in the prompt to set resolution (Gemini):** cosmetic only — sets the *look*, not pixels. Use `image_size`.
- **Contradictions:** `hyperrealistic photo` + `flat 2D cartoon`, or `close-up` + `full body wide shot`. One medium, one framing per prompt.
- **Vague adjectives:** beautiful, nice, stunning, aesthetic, amazing — zero visual information. Replace with material/light/colour/framing.
- **Kitchen-sink first draft:** start short and single-purpose, then iterate. Easier to debug what broke.
- **Naming a real person on Gemini:** refused since the Feb 2026 tightening. Describe generically instead.

---

## Pre-send checklist

Run this before emitting any prompt.

1. **Engine chosen** (or portable prompt + per-engine config given)?
2. **Natural sentences, ordered** background → subject → details → style → constraints? (Not comma tags, not MJ flags.)
3. **One medium, one framing** — no contradictions?
4. **Every adjective renderable** — swapped "nice/beautiful/epic" for material, light, colour, lens, framing?
5. **Aspect ratio in words** + the right config (`size` for ChatGPT; `aspect_ratio` + `image_size` for Gemini; "4K" in prompt is cosmetic)?
6. **In-image text** quoted verbatim, marked "EXACT", typography + placement specified, bounded to appear once; `quality:"high"` (ChatGPT) or Nano Banana Pro (Gemini) if dense?
7. **Exclusions phrased positively**, any hard "no X" placed last?
8. **If an edit:** one change only, full preserve-list restated this turn; Responses API (ChatGPT) or Interactions API (Gemini); `input_fidelity:"high"` for faces/logos on ChatGPT?
9. **Right model?** ChatGPT: gpt-image-2 default, gpt-image-1.5 for transparent background, gpt-image-1-mini for cheap drafts. Gemini: Nano Banana 2 default, Nano Banana Pro for text/identity, Lite for speed.
10. **Policy sane?** No identifiable real person on Gemini; no sexual/violent/misleading real-person depiction on ChatGPT; both are SynthID-watermarked.
