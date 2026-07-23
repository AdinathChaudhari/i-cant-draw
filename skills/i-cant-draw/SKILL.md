---
name: i-cant-draw
description: Turn a rough image idea into a finished, ready-to-paste image-generation prompt for the engine the user picks — ChatGPT (OpenAI GPT Image) or Google Gemini (Nano Banana). Use this skill WHENEVER you are about to generate, request, or edit an image, write an image prompt, or help a user create/edit/restyle a picture, logo, poster, icon, infographic, product shot, or character. The flow is: pick the model, describe the scene, and this skill produces the prompt (plus that model's config) built from a self-contained, engine-specific guide — so the two engines' rules never conflict. Trigger even when the user just says "make me an image of…", "draw…", or "generate a picture of…".
---

# i-cant-draw

You produce a finished image-generation prompt for **one** engine the user selects. You do not generate the image; you generate the *prompt* (plus the config for that engine) and hand it over — or call the image tool if you have one.

Each engine has its own self-contained guide. You load exactly one. Nothing from the other engine's guide applies, so there are no conflicting rules to reconcile.

## The flow

1. **Model — resolve this first.** Which engine: **ChatGPT** (OpenAI GPT Image) or **Gemini** (Google Nano Banana)?
   - If the user named one, use it.
   - If not, ask one short question: *"ChatGPT or Gemini?"* Do not proceed on a guess — the two need different config (aspect ratio, model IDs, editing API), and picking wrong wastes the prompt.
2. **Scene — get the intent.** What is the image (subject, purpose, any must-haves like text, colours, orientation)? Take whatever the user gives; you will enrich the vague parts using the engine guide. Ask only if you cannot tell the core subject.
3. **Generate — build from the engine's guide.** Read the matching file, then write the prompt by following it exactly:
   - **ChatGPT →** read `references/chatgpt.md`
   - **Gemini →** read `references/gemini.md`

## Universal rules (true for both engines)

These hold regardless of engine — both OpenAI and Google state them explicitly. Apply them always; the engine guide adds the model-specific config on top.

- **Describe the scene in natural-language sentences; don't list keywords.** A narrative sentence beats a comma-salad of tags. This is the opposite of Midjourney/Stable Diffusion habits.
- **Be specific.** Underspecified prompts don't fail — they get silently filled with the model's guesses. Name anything you care about, with concrete material/light/colour/framing detail, not vague adjectives ("beautiful", "8K", "epic").
- **Order it the way the image is built:** background/scene → subject → action → composition/framing → style/medium → lighting → colour → mood → constraints (last). State the intended use early.
- **One medium, one framing** per prompt. No "photorealistic" + "flat cartoon"; no "close-up" + "wide shot".
- **Phrase exclusions positively.** Neither engine has a working negative-prompt field, and naming a thing to exclude tends to summon it. Say "an empty street", not "no cars". Keep any "no watermark, no extra text" as a short trailing clause.
- **No Midjourney/SD syntax** (`--ar`, `--v`, `::2`, `(word:1.5)`, seeds) — ignored or rendered as literal text by both.
- **Quote in-image text verbatim**, keep it short, and specify placement and font *style*.
- **Edit in small deltas.** One change per turn, and restate what must stay the same.

## Output shape

When you generate, emit exactly this:

1. **The prompt** — natural-language, ready to paste into the chosen tool.
2. **Config** — a short block of the engine's real settings (model, size/aspect ratio, quality, etc.), taken from that engine's guide.
3. *(Optional)* one **Refine** hint — a follow-up phrasing the user can send to iterate.

Do not include the other engine's config. Do not explain the rules back to the user unless asked — just deliver the prompt.

## Then what

Open the engine guide for the full formula, the dimension vocabulary, text handling, editing and consistency workflow, API/config reference, anti-patterns, content policy, worked examples, and the pre-send checklist. Follow that guide; this file is only the router.
