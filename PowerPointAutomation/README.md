# VPS PowerPoint Template Bundle (Minimal)

This folder is the **minimum upload bundle** for agents to build decks from the 4 template libraries.

## Included Files

- `template_catalog.json` - global index of all template IDs (`template_id -> deck_id + slide_number + placeholders`)
- `templates/<deck_id>/template_deck.pptx` - source template deck for that library
- `templates/<deck_id>/manifest.json` - per-slide metadata + required/optional placeholders
- `templates/<deck_id>/payload.json` - blank payload skeleton for all slides in that deck

Not included on purpose:
- screenshots
- reverse-engineering artifacts
- extraction scripts

---

## Agent Contract (How to Build Slides)

### 1. Choose a template library

Use `template_catalog.json` to select `deck_id` and `template_id` values.

### 2. Build an input job

Use this structure:

```json
{
  "deck_id": "Moskva_powerpoint",
  "output_pptx": "/tmp/output.pptx",
  "slides": [
    {
      "template_id": "Moskva_powerpoint-S01",
      "values": {
        "{SlideMainExplanation}": "Your heading",
        "{SlideSupportingContent}": "Your subtext",
        "{SlideImage}": "/tmp/hero.jpg"
      }
    }
  ]
}
```

### 3. Compose the output deck

- Open `templates/<deck_id>/template_deck.pptx`.
- For each requested slide, locate `slide_number` from `template_catalog.json`.
- Duplicate that source slide into a new output deck in requested order.
- Replace only known `{Slide...}` tokens from `values`.

### 4. Image insertion rules

- These templates preserve native PowerPoint image placeholders (`p:ph type="pic"`) where they exist.
- If a token value is an image path/URL, insert into matching image slot order:
  - first image token -> primary image slot (`{SlideImage}`)
  - second image token -> next slot (`{SlideSupportingImage}`)
  - then `{SlideSupportingImage2}`, etc.
- Some slides use normal image objects (not click-to-insert placeholders). For those, replace the placeholder image object content while keeping frame/mask/crop unchanged.

### 5. Preserve design system (non-negotiable)

- Do not change theme, master, layout, fonts, spacing, colors, masks, or decorative elements.
- Do not rename placeholders.
- Do not remove image masks/crops.

### 6. Validate before returning

Fail the job if any are true:
- unresolved `{Slide...}` tokens remain
- missing required placeholders from manifest
- wrong template library (`deck_id`) used
- output slide count differs from requested slide count

---

## Fast Lookup

- Deck list: `template_catalog.json -> decks[]`
- Template lookup: `template_catalog.json -> template_index[template_id]`
- Slide details: `templates/<deck_id>/manifest.json`

---

## Recommended Output Artifacts

For each agent run, return:
- `output.pptx`
- `run_report.json` with:
  - `deck_id`
  - `template_ids_used`
  - `slides_requested`
  - `slides_rendered`
  - `unresolved_tokens` (array)
  - `missing_required_placeholders` (array)
  - `status` (`pass`/`fail`)
