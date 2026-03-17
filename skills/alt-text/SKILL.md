---
name: alt-text
description: Write concise alt text for a local image file. Use this when the user asks for alt text, accessibility text, or a brief image description for a specific local image path. Pass the target image path as `$0`.
---

# alt-text

## Input
The target local image path is provided as `$0` or as a pasted image attachment.

If `$0` is missing, ask the user for the local image path and stop.

## Instructions
When this skill is invoked, inspect the image directly and write alt text immediately.

- Describe what is visually present and important.
- Do not say `picture of`, `image of`, `screenshot of`, or `diagram of`.
- Describe what the image shows, not the file type or medium.
- Keep the alt text concise and focused on the essential content.
- Do not add extraneous details, speculation, backstory, or interpretation that is not clearly visible.
- If visible text is important to understanding the image, include the key text briefly.

## Output
Return only the alt text unless the user asks for something else.
