# Prompt Cookbook — i-cant-draw

Copy-pasteable library for writing OpenAI/ChatGPT image prompts. Companion to `SKILL.md`. Assume **gpt-image-2** (ChatGPT's default as of 2026) unless the user is in the legacy DALL-E GPT.

**The one-paragraph model of how these models work:** gpt-image-1/1.5/2 and GPT-4o are *autoregressive* — they build the image as a token sequence (roughly background → foreground) and reason over your words semantically. That is why they follow instructions, render real text, and edit conversationally, unlike DALL-E 3 diffusion. So: write ordered natural-language sentences (never keyword soup), state exclusions last, and describe what you *want* — not what you don't.

---

## Part 1 — Weak vs. Strong prompt pairs

Eleven pairs across the common use cases. Steal the structure, swap the nouns.

### 1. Photorealistic scene

**Weak:** `a nice photo of a person looking happy`

**Strong:**
> Photorealistic candid photograph of an elderly sailor on a fishing boat, weathered skin with visible pores and stubble, adjusting a rope net with both hands. Overcast harbor behind him. Shot on 35mm film, soft diffuse daylight, shallow depth of field, natural color grading, subtle film grain. Wide 3:2 landscape. No glamorization, no text.

*Why it wins:* concrete texture + camera language ("visible pores," "35mm film," "shallow depth of field") triggers realism mode far harder than "nice" or "8K."

### 2. Illustration / stylized art

**Weak:** `cool fantasy forest, epic, trending, 8k, detailed`

**Strong:**
> Flat cel-shaded digital illustration of a moss-covered forest clearing at dusk. A single deer silhouette drinks from a still pond in the foreground. Warm bioluminescent mushrooms are the only light source. Muted teal-and-amber palette, soft painterly shadows, storybook mood. Vertical 2:3 portrait.

*Why it wins:* one explicit medium ("flat cel-shaded illustration") suppresses unwanted photorealism; a named palette and single focal subject stop the muddled blend "epic/trending/8k" produces.

### 3. Logo / graphic WITH text

**Weak:** `make a cool logo for my bakery called Field & Flour`

**Strong:**
> Original, non-infringing logo for a bakery. Wordmark text (EXACT, verbatim): "FIELD & FLOUR". A strong single wheat-stalk silhouette above the text, clean flat vector shapes, balanced negative space, minimal strokes, no gradients. Warm brown and cream palette. Typography: rounded serif, centered below the icon, generous padding. Plain white background, single centered logo. Reads clearly at small and large sizes. No watermark, no extra text.

*Why it wins:* text is quoted + marked verbatim, typography is a hard constraint, and "reads clearly at small sizes / flat vector / minimal strokes" forces the simplicity a real logo needs.

### 4. Product shot

**Weak:** `product photo of a water bottle, professional`

**Strong:**
> Studio e-commerce product photograph of a matte black 750ml aluminum water bottle, centered on a seamless light-gray background. Soft three-point studio lighting with a gentle top highlight and a subtle reflection beneath the bottle. Sharp focus, true-to-life color. No props, no hands, no text. Square 1:1.

*Why it wins:* names the exact studio setup and background that catalog shots actually use; excludes props/text that clutter listing images.

### 5. Character portrait

**Weak:** `portrait of a cyberpunk woman, badass, detailed, 4k`

**Strong:**
> Photorealistic close-up character portrait of a woman in her 30s with short silver hair and a faint neon-blue circuit tattoo along her jaw, wearing a matte black tactical jacket, looking directly at camera. Neon magenta and cyan rim lighting against a dark rain-slicked alley background. Shot as if on an 85mm portrait lens, shallow depth of field. Vertical 2:3.

*Why it wins:* every "badass" is replaced with a renderable detail (silver hair, circuit tattoo, rim lighting); the 85mm + shallow DoF gives a real portrait look.

### 6. Infographic / diagram (text-heavy → bump quality)

**Weak:** `infographic about coffee, informative, clean`

**Strong:**
> Clean infographic titled "HOW ESPRESSO IS MADE", portrait orientation. Five numbered stages left to right, each a simple labeled icon connected by arrows. Labels (EXACT, verbatim): "1 GRIND", "2 DOSE", "3 TAMP", "4 BREW", "5 POUR". White background, single espresso-brown accent color used consistently, clear visual hierarchy, legible sans-serif labels. Verify exactly five stages and no extra text. Avoid tiny text or decoration that hurts readability.

*Why it wins:* every label is listed explicitly and bounded ("exactly five"); this is the case where you also set `quality:"high"` in the API. Dense multi-label work is the reliability edge — fewer labels per image = fewer garbles.

### 7. UI mockup

**Weak:** `app design for farmers market, modern UI`

**Strong:**
> Realistic mobile app UI mockup shown inside an iPhone frame. Top: header with market name and a search icon. Middle: a "TODAY'S SPECIALS" banner, then a scrollable vendor list with square photos and vendor names. Bottom: a tab bar with Home, Map, and Profile icons. White background, green-and-orange accent colors, clean sans-serif typography, generous spacing. Vertical 2:3.

*Why it wins:* lays out the screen region-by-region (top/middle/bottom) so the model places elements instead of inventing a generic "modern" blob.

### 8. Poster

**Weak:** `movie poster for a thriller`

**Strong:**
> Cinematic movie poster for a psychological thriller. Title text (EXACT, verbatim): "THE LONG DARK", set in bold condensed uppercase at the bottom third, with empty space beneath for credits. A lone figure stands at the end of a fog-filled street, backlit by a single sodium streetlight. High-contrast moody lighting, desaturated blue palette with one warm highlight. Vertical 2:3. Ensure the title appears once, legible. No other text, no watermark.

*Why it wins:* separates the display text (quoted, placed, bounded to once) from the pictorial description, and reserves credit space the way real key art does.

### 9. Icon / asset with transparent background

**Weak:** `an otter icon`

**Strong:**
> Minimalist otter icon, flat vector style, single centered shape, bold clean outlines, soft rounded forms, friendly. Two-color fill (warm brown + cream), no background elements, no text. Square 1:1.

*API note:* set `background:"transparent"`, `output_format:"png"`, and use `gpt-image-1.5` — **gpt-image-2 dropped transparency support.**

*Why it wins:* "single centered shape / flat vector / no background elements" keeps it icon-clean; transparency is an API flag, not a prompt word.

### 10. Multilingual / non-Latin text

**Weak:** `a Japanese festival poster with some Japanese text`

**Strong:**
> Festival poster in a woodblock-print style. Headline text (EXACT, verbatim, Japanese): "夏祭り". Below it in smaller type (EXACT): "2026". Lanterns and fireworks over a river at night, warm red-and-gold palette. Headline centered at top, large and legible. Vertical 2:3. No other text.

*Why it wins:* non-Latin scripts (Japanese, Korean, Chinese, Hindi, Bengali) only became reliable in gpt-image-2 — still quote the exact glyphs and keep the string short.

### 11. Iterative edit / localization (constraint-heavy)

**Weak:** `make it in Spanish, keep it looking the same-ish`

**Strong:**
> Translate all visible text in this image to Spanish, verbatim and accurate. Change only the text. Keep everything else pixel-identical: layout, typography style, spacing, text hierarchy, color palette, logo position, and background imagery. Do not reflow unless a translated string genuinely does not fit.

*Why it wins:* an edit succeeds or drifts based on the *preserve list* — spell it out, every turn (see Part 5).

---

## Part 2 — Reusable template formulas

Fill the brackets. Keep the ordering; put exclusions last.

### Template A — Photoreal scene
```
Photorealistic [shot type] of [subject + defining physical details]
[doing action], in [setting/background].
[Lighting: e.g. soft overcast daylight / golden-hour side light].
Shot on [lens, e.g. 35mm film / 85mm portrait lens], [depth-of-field cue].
[Color grading]. [Texture callouts: pores, fabric wear, grain].
[Aspect ratio in words]. [Exclusions: no text, no watermark].
```

### Template B — Illustration / stylized
```
[Explicit medium: flat vector / watercolor / cel-shaded / line art]
illustration of [subject + action], in [setting].
[Named color palette]. [Light source]. [Mood].
[Composition + focal subject]. [Aspect ratio]. [Exclusions].
```

### Template C — Graphic / logo WITH text
```
[Graphic type] for [entity/purpose].
[Text role] text (EXACT, verbatim): "[EXACT STRING]".
[Icon/imagery description]. [Style: flat vector / minimal strokes].
[Named palette]. Typography: [font style, weight, case], [placement],
[size relationship to icon]. [Background treatment].
Ensure text appears once and is perfectly legible.
No watermark, no extra text.
```

### Template D — Product shot
```
Studio product photograph of [product + material + size], centered on
[seamless background color]. [Lighting setup, e.g. soft three-point with top highlight].
[Reflection/shadow treatment]. Sharp focus, true-to-life color.
For [use, e.g. an e-commerce listing]. [Aspect ratio].
No props, no hands, no text.
```

### Template E — Iterative edit
```
[Single change: "change only X to Y"].
Keep everything else identical — [explicit list: layout, colors,
camera angle, background, identity/face, pose, logo position, typography].
[Restate this whole preserve-list on EVERY follow-up turn.]
```

---

## Part 3 — Vocabulary cheat-sheet

Words that actually steer OpenAI models. Prefer these over `8K` / `masterpiece` / `trending on artstation` (SD-era tags, largely ignored).

| Dimension | Words that steer the model |
|---|---|
| **Composition / framing** | close-up, medium close-up, medium shot, wide shot, extreme wide, top-down/overhead, over-the-shoulder, rule-of-thirds, centered, negative space on [side], symmetrical, foreground/midground/background, tight crop |
| **Camera angle** | eye-level, low-angle, high-angle, worm's-eye, bird's-eye, Dutch tilt, straight-on, three-quarter view, profile |
| **Camera / lens** | 24mm wide lens, 35mm film, 50mm, 85mm portrait lens, macro, shallow depth of field, deep focus, bokeh background, motion blur, long exposure, film grain, lens flare |
| **Lighting** | soft diffuse light, golden hour, blue hour, high-contrast, harsh overhead, rim lighting, backlit, side light, natural window light, studio three-point, neon glow, chiaroscuro, overcast |
| **Art style / medium** | photorealistic, 35mm film photograph, flat vector illustration, watercolor, gouache, cel-shaded, line art, ink drawing, 3D render, isometric, pixel art, oil painting, charcoal sketch, woodblock print, claymation |
| **Color palette** | warm autumnal palette, cool blue-and-teal tones, muted pastels, high-saturation neon, monochrome black-and-white, earth tones, desaturated, complementary orange-and-blue, natural color balance |
| **Mood** | serene/calm, tense, cinematic, whimsical, melancholic, energetic, ominous, cozy, minimalist, nostalgic, dramatic, playful |

**Realism dial:** to push realism, add texture cues (visible pores, skin wrinkles, fabric wear, worn metal, everyday imperfections). To suppress it, lead with an explicit illustration medium (flat vector, watercolor wash, cel-shaded).

---

## Part 4 — gpt-image API quick reference

Two endpoints. Both return `b64_json` (never a hosted URL — decode it yourself).

- **Generate:** `POST /v1/images/generations` → `client.images.generate(...)` — prompt only.
- **Edit:** `POST /v1/images/edits` → `client.images.edit(...)` — 1+ input images (up to 16) + optional mask + prompt.

### Key parameters

| Param | Values | Notes |
|---|---|---|
| `model` | `gpt-image-2` (flagship) · `gpt-image-1.5` (default, transparency) · `gpt-image-1-mini` (cheap drafts) · `gpt-image-1` (deprecated) | Must set explicitly, else defaults to `dall-e-2`. |
| `prompt` | string, up to 32,000 chars | Fully self-contained — the API has **zero memory** between calls. |
| `n` | 1–10 | Use `n=4` to pick the cleanest text/logo render. |
| `size` | `1024x1024`, `1536x1024`, `1024x1536`, `auto` (v1/1.5); gpt-image-2 also takes arbitrary `WxH` up to 3840px edge, multiples of 16, ratio ≤ 3:1 | Above 2560x1440 is experimental. Still state aspect ratio in the prompt too. |
| `quality` | `low` / `medium` / `high` / `auto` | **Biggest lever for text fidelity.** Use `high` for text/infographics/portraits. |
| `background` | `transparent` / `opaque` / `auto` | Transparent needs `output_format` png/webp AND `gpt-image-1`/`1.5` — **not gpt-image-2**. |
| `output_format` | `png` / `jpeg` / `webp` | jpeg has no alpha. |
| `moderation` | `auto` / `low` | Only two levels. |
| `input_fidelity` | `low` / `high` (edit only, v1/1.5) | `high` preserves faces/logos/fine detail. Use for identity-sensitive edits. |
| `mask` | PNG, <4MB, same dimensions as input | Transparent alpha = region to regenerate. **Loose, not precise** — confirmed limitation on gpt-image-1/2; use explicit "change only X" text, or dall-e-2 for true masked inpainting. |

### Minimal generate request (Python)
```python
import base64
from openai import OpenAI
client = OpenAI()

r = client.images.generate(
    model="gpt-image-2",
    prompt=(
        'Studio product photograph of a matte black aluminum water bottle, '
        'centered on a seamless light-gray background, soft three-point lighting, '
        'subtle reflection beneath. For an e-commerce listing. Square 1:1. '
        'No props, no text.'
    ),
    size="1024x1024",
    quality="high",
    n=1,
)
open("bottle.png", "wb").write(base64.b64decode(r.data[0].b64_json))
```

### Minimal edit request (curl)
```bash
curl https://api.openai.com/v1/images/edits \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -F "model=gpt-image-2" \
  -F "image[]=@poster.png" \
  -F "prompt=Change only the sky to a sunset gradient; keep the building, people, and foreground pixel-identical, same layout and color grading." \
  -F "quality=high" \
  -F "input_fidelity=high"
```

### Multi-turn editing (Responses API)
```python
r1 = client.responses.create(model="gpt-5.6",
    input="Generate a gray tabby cat hugging an otter, orange scarf",
    tools=[{"type": "image_generation"}])

r2 = client.responses.create(model="gpt-5.6",
    previous_response_id=r1.id,
    input="Change only the scarf to red; keep the cat, otter, pose, and background identical.",
    tools=[{"type": "image_generation", "action": "edit"}])
```
`action`: `"auto"` (model decides) · `"generate"` (force new) · `"edit"` (force modify).

**Rough output cost** (gpt-image-2, per image): low ≈ $0.01, medium ≈ $0.04, high portrait/landscape ≈ $0.25. Reference-image edits bill inputs at high-fidelity rate.

---

## Part 5 — Iterative-editing phrasebook

**Golden rule:** stay in one thread, change one thing per turn, and restate the preserve-list every time. Nothing persists reliably — drift compounds.

**Phrase edits mechanically, not by vibe:**

| Instead of (vague) | Say (mechanical) |
|---|---|
| make it warmer | increase warmth in the color grading, add a slight orange/gold tint to the lighting |
| make it darker | increase the shadows and slightly reduce overall brightness |
| make it pop | increase saturation and add stronger rim lighting on the subject |
| make it cleaner | remove background clutter, increase negative space, simplify to one accent color |
| better composition | recompose to rule-of-thirds with the subject offset left, more headroom |

**Reliable edit instructions:**
- Reframe: `zoom out to show the full body` · `crop tighter to a head-and-shoulders close-up` · `extend the scene to the left`
- Add/remove: `add a small dog sitting in the background; keep everything else the same` · `remove the coffee cup on the table, fill with matching wood grain`
- Recolor one thing: `change only the jacket to deep red; keep the face, pose, and background pixel-identical`
- Time/weather: `same scene at night, neon signs lit; keep the layout and subject identical`

**Preserve-list to append (pick what matters):**
> Keep everything else identical: layout, camera angle, subject's face and identity, pose, outfit, background, color palette, logo position, typography.

**Consistency across a series (no seed exists):**
1. Make one clean, front-facing **canon reference** image.
2. Upload it every time: `use this character as reference, keep the same face and outfit`.
3. Reuse the **exact same** character description and style phrase — never paraphrase (synonym drift = visual drift).
4. Chain forward: each new image references the previous output, changing only scene/action. Don't batch independent variants.
5. Don't switch tools mid-project — different models render the same prompt differently.

---

## Part 6 — Anti-patterns (delete these on sight)

- **Midjourney/SD flags:** `--ar 16:9`, `--v 6`, `--stylize`, `::2`, `(word:1.5)`, `--seed`. All ignored or rendered as literal text. Use plain English + the `size` param.
- **Keyword soup:** `cyberpunk woman, neon, 8k, ultra detailed, trending, octane render`. Reads as noise. Write a sentence.
- **Negative prompts as the focus:** `an elephant without a fedora` → tends to render the fedora. Describe the positive scene (`a bare-headed elephant`). Short single exclusions are fine only as a trailing constraint.
- **Contradictions:** `hyperrealistic photo` + `flat 2D cartoon`, or `close-up` + `full body wide shot`. One medium, one framing per prompt.
- **Vague adjectives:** beautiful, nice, stunning, aesthetic, amazing — zero visual information. Replace with material/light/color/framing.
- **Kitchen-sink first draft:** start short and single-purpose, then iterate. Easier to debug what broke.

---

## Pre-send checklist

Run this before emitting any prompt.

1. **Natural sentences, ordered** background → subject → details → style → constraints? (Not comma tags, not MJ flags.)
2. **One medium, one framing** — no contradictions?
3. **Every adjective renderable** — swapped "nice/beautiful/epic" for material, light, color, lens, framing?
4. **Aspect ratio stated in words** (and `size` set if calling the API)?
5. **In-image text quoted verbatim**, marked "EXACT," typography + placement specified, bounded to appear once, tricky words spelled out?
6. **Exclusions phrased positively where possible**, and any hard "no X" placed **last**?
7. **Text-heavy or small type?** → set `quality:"high"`; list every label explicitly; consider splitting dense diagrams into panels.
8. **If an edit:** one change only, full preserve-list restated this turn, `input_fidelity:"high"` for faces/logos?
9. **Right model?** gpt-image-2 default; gpt-image-1.5 for transparent background; gpt-image-1-mini for cheap drafts; DALL-E 3 only if the user explicitly wants its diffusion aesthetic.
10. **Policy sane?** No sexual/violent/misleading depiction of a real person (public figures otherwise OK, but some are on OpenAI's opt-out list and will refuse regardless).
