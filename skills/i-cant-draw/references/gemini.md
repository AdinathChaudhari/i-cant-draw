# Google Gemini / Nano Banana — engine guide

Self-contained guide for writing prompts for **Google Gemini** image generation ("Nano Banana") — the Gemini app and the Gemini API. Load this only when the user chose Gemini. Everything here is Gemini-specific; ignore ChatGPT's rules.

## Models

| Model | Nickname | Use it for |
|---|---|---|
| `gemini-3.1-flash-image` | Nano Banana 2 | Default. General workhorse, up to 4K, best speed/consistency balance. What the Gemini app uses. |
| `gemini-3-pro-image` | Nano Banana Pro | Premium. Best text rendering (paragraph-scale) and identity/brand consistency; reference-heavy work. |
| `gemini-3.1-flash-lite-image` | Nano Banana 2 Lite | Cheapest/fastest. **Not** for multi-reference or multi-turn editing. |
| `gemini-2.5-flash-image` | Nano Banana (legacy) | Migrate off it. |
| Imagen 4 | — | Being shut down in 2026. Do not target it for new work. |

Nano Banana models are native, conversational, multimodal: text and image in one turn, image inputs for edit/compose/reference, and a reasoning step before drawing. Write full sentences.

## The prompt formula

```
[Medium/style] of [subject with defining details] [action/pose], in [setting/background].
[Framing: shot type + camera angle + lens]. [Lighting]. [Colour palette]. [Mood]. For [intended use].
[Constraints last: exclusions, aspect ratio in words.]
```

For complex scenes, either use labelled segments (`Scene:`, `Subject:`, `Style:`, `Text (verbatim):`, `Constraints:`) or give **step-by-step** instructions, which Google recommends: `First, create [background]. Then add [midground]. Finally, place [foreground element].`

## Dimension vocabulary

- **Subject/action:** be hyper-specific. `ornate elven plate armour etched with silver leaf, high collar, pauldrons shaped like falcon wings`, not `fantasy armour`.
- **Style/medium:** say `photorealistic` + camera words (`35mm film`, `85mm lens`, `shallow depth of field`, `film grain`) for realism; lead with an explicit medium (`flat vector`, `watercolour`, `cel-shaded`, `kawaii sticker`) for stylised work.
- **Composition/framing:** shot type + angle + lens + explicit placement (`positioned in the bottom-right of the frame`, `vast empty off-white canvas` for text-overlay space).
- **Lighting:** `natural`/`dramatic`/`warm`/`cold`; named setups — `sunbeams filtering through`, `three-point softbox`, `studio-lit`.
- **Colour:** named palette — `muted orange warm tones`, `cool blue-and-teal`.
- **Mood:** pair with colour/lighting in the same clause.

Google documents named templates you can lean on: photorealistic scene, kawaii sticker/logo, text-heavy design, product mockup, minimalist/negative-space, and sequential comic panels.

## In-image text (use Nano Banana Pro)

- Quote the exact text in quotation marks; describe font style descriptively (`clean, bold, sans-serif`), not by brand name.
- State placement/layout and overall design intent explicitly.
- **Text-first trick:** state the exact text content first, then describe the image around it — more reliable than asking for both at once.
- Use **`gemini-3-pro-image` (Nano Banana Pro)** for anything text-heavy; it handles paragraph-scale text and infographics (keep total in-image text under ~400 words). The Flash tiers are weaker on dense text.

## Editing and consistency (Gemini's headline strength)

- **Conversational editing is the recommended way to iterate.** Use the **Interactions API** (`previous_interaction_id`) so you don't resend the image each turn.
- Append a preserve clause to **every** edit turn — Google's own examples say **"Do not change any other elements of the image."** Example: `That's great, but make the lighting a bit warmer. Do not change any other elements.`
- **Localised edits use semantic masking — no mask file.** `Using the provided image, change only the blue sofa to a brown leather chesterfield. Keep everything else exactly the same, preserving the style, lighting, and composition.`
- **Reference images are split by role** (up to 14 total on Nano Banana 2/Pro): object fidelity (~10), character consistency (~4–5), style reference (~3). Reference each by position — "the first image", "the second image".
- **Multi-image fusion:** `Take the [element] from the first image and place it with the [element] from the second image. The final image should be [scene].`
- **Character consistency:** upload character-role images, reuse the exact same description, and say `Ensure the face and features remain completely unchanged`. For turnarounds, feed each generated angle back in as a reference for the next.
- **Quality degrades after ~3–5 chained edits** (each edit works on the previous output, like re-saving a JPEG). Once you like a result, re-derive a full descriptive prompt and regenerate fresh.

## Config / API reference

Two surfaces: `generateContent` (legacy, fully supported) and the **Interactions API** (recommended, threaded via `previous_interaction_id`). Every output carries a non-optional **SynthID** watermark. There is **no negative-prompt field** and **no per-request moderation dial**.

| Config | Values | Notes |
|---|---|---|
| `model` | `gemini-3.1-flash-image` · `gemini-3-pro-image` · `gemini-3.1-flash-lite-image` | Pro for dense text / identity. |
| `response_modalities` | `["IMAGE"]` | Required for image output on `generateContent`. |
| `image_config.aspect_ratio` | `1:1, 3:2, 2:3, 3:4, 4:3, 4:5, 5:4, 9:16, 16:9, 21:9` | An **enum**, not pixels. Omit → inferred from the input image. |
| `image_config.image_size` | `"1K"`, `"2K"`, `"4K"` | **Uppercase K required.** This sets real resolution — writing "4K" in the prompt text is cosmetic only. |
| reference images | up to 14, split by role | object (~10) / character (~4–5) / style (~3). |

```python
from google import genai
from google.genai import types
client = genai.Client()
r = client.models.generate_content(
    model="gemini-3.1-flash-image",
    contents=("Studio product photograph of a matte black aluminium water bottle, centred on a "
              "seamless light-grey background, soft three-point lighting, subtle reflection beneath. "
              "For an e-commerce listing. Square. No props, no text."),
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

## Anti-patterns

- Midjourney/SD flags (`--ar`, `::2`) → not parsed. Put the ratio in `image_config.aspect_ratio`; put emphasis in prose ("the *only* light source is…").
- Keyword soup → Google says "describe the scene, don't list keywords." Write sentences.
- Negative prompts → no field; use positive/semantic framing (`an empty, deserted street with no signs of traffic`, not `no cars`).
- "4K"/"HD" in the prompt to set resolution → cosmetic; use `image_size`.
- Vague adjectives → be hyper-specific about material, lens, angle, lighting.
- Overloading one prompt → iterate conversationally, one change per turn.

## Content policy

- **Real people:** Gemini **refuses identifiable real named people** (celebrities, politicians, anyone named) — tightened hard in the February 2026 safety upgrade, not bypassable via settings. Rephrase generically: `a woman in her 30s in business attire`.
- **Allowed:** fictional characters, stylised/illustrated people, generic non-identifiable figures.
- **Copyrighted characters/trademarked logos:** expect refusals or generic substitutions.
- **Uploaded references:** you must have the rights to them.
- **Hard blocks:** sexual content involving minors, non-consensual intimate imagery, graphic gore, self-harm how-to, hate/harassment imagery.

## Worked examples (weak → strong)

**Photoreal:** `a nice photo of a person looking happy` →
> Photorealistic candid photograph of an elderly sailor on a fishing boat, weathered skin with visible pores, adjusting a rope net. Overcast harbour behind him. Shot on 35mm film, soft diffuse daylight, shallow depth of field, subtle film grain. No text. Config: `aspect_ratio="3:2"`, `image_size="2K"`.

**Logo with text (use Pro):** `cool logo for my bakery Field & Flour` →
> Original, non-infringing logo for a bakery. Wordmark text (EXACT, verbatim): "FIELD & FLOUR", in a clean bold serif, centred below the icon. A single wheat-stalk silhouette above the text, flat vector, minimal strokes, no gradients. Warm brown and cream palette, plain white background. Config: `model="gemini-3-pro-image"`, `aspect_ratio="1:1"`, `image_size="2K"`.

**Multi-image fusion:** `put my dress on the model` →
> Create a professional e-commerce fashion photo. Take the blue floral dress from the first image and let the woman from the second image wear it. Generate a realistic full-body shot with lighting and shadows matching an outdoor golden-hour environment. Keep her face and the dress pattern unchanged. Config: pass both images as input parts; `aspect_ratio="4:5"`.

## Pre-send checklist

1. Natural sentences, ordered, intended use stated — not tags or flags?
2. One medium, one framing?
3. Every adjective renderable (material/light/colour/framing), hyper-specific?
4. Aspect ratio in words **and** `image_config.aspect_ratio` set; `image_size` chosen (remember "4K" in the prompt is cosmetic)?
5. In-image text quoted verbatim, short, typography described; using Nano Banana Pro if dense?
6. Exclusions phrased positively (semantic negatives), any "no X" placed last?
7. If editing: one change + "do not change any other elements" this turn; Interactions API with `previous_interaction_id`?
8. Consistency work: same description reused verbatim; references passed by role (object/character/style)?
9. Right model (Nano Banana 2 default; Pro for text/identity; Lite for speed)?
10. Policy-safe (no identifiable real named person; nothing hard-blocked)?

Output the prompt + a Config block; optionally one Refine hint.
