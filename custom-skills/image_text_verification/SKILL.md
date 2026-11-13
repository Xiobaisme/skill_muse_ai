---
name: image-text-verification
description: Verify text in AI-generated images before delivery. Plans the exact headline and line breaks before generation, transcribes the rendered image word by word, and regenerates until the text matches the approved headline and the user's line rules.
---

# Verify Generated Image Text

## Plan the exact text before generation

- Copy the headline exactly as the user wrote it into the generation brief, including spelling, numbers, punctuation and symbols.
- Break the headline into lines before generation and check each line against the user's line rules, including maximum line length and any rule against a line holding only one word.
- Name one hero object in the brief and describe only that object, so the image keeps a single visual focus.
- Carry the user's stated ratio, palette, font and layout zones into the brief without adding text, logos or decorative elements the user did not ask for.

## Inspect the rendered image

- Read the generated image and transcribe every visible word exactly as rendered, including small and focal lines.
- Compare the transcription word by word with the approved headline and flag any dropped word, added word, misspelling or changed number or symbol.
- Check each rendered line against the approved line breaks and flag a line that holds only one word when the user forbade that pattern.
- Check the hero count, the layout zones and the forbidden elements list, and flag any extra object, logo, brand mark or added text.

## Fix before delivery

- Regenerate the image with a corrected brief when the inspection finds a defect, and repeat the transcription check on the new image.
- Present an image as finished only after its transcription matches the approved headline and its lines meet the user's line rules.
- When a defect cannot be fixed by regeneration, tell the user the exact defect and the corrected line split you propose, and wait for approval before presenting a revised image.
- When the user approves a line split in advance, use that split unchanged in every regeneration for that poster.

## Available connectors

- Use `muse.connector_search` to find other connectors and `muse.skill_search` to find related skills.
- If nothing here fits, look on the web for a maintained API or MCP server and set it up with Custom Connectors and MCPs, or build a custom connector.

## Alternatives for better results

- Ask the user to approve the line split before generation when a headline is long or hard to break, since an approved split removes a common cause of regeneration.
- Ask for a shorter headline or a smaller focal phrase when the exact headline cannot fit the stated layout without breaking a line rule.
