---
name: source-backed-changes
description: Create or extend visual assets from exact references; modify source-backed data or media; or verify the identity, compatibility, and served form of a requested artifact.
---

# Source-backed changes

Use this skill when the requested result depends on an existing source artifact or
on the exact identity of a visible record. Keep the checks below focused on that
artifact; ordinary implementation, build, and deployment rules come from the project.

- Inspect the exact user-provided page, record, asset, screenshot, recording, log, or artifact first.
- Before creating or extending visual assets, open representative original assets and inspect their actual rendered
  context. Record the relevant dimensions, pixel scale, palette, outlines, shading, perspective, and relative size;
  use these as acceptance criteria unless the user requests a new visual direction.
- Reuse suitable existing assets. When generation is needed, supply the inspected originals as image references,
  together with the concrete constraints; a genre label such as pixel art is not a substitute for reference images.
- Validate a small representative sample beside the originals at the intended display scale before producing a batch.
  Correct mismatched style or scale before expanding it; reducing display size alone does not establish a match.
- For UI complaints, identify the exact visible view, component, toolbar, and state before editing
- Preserve the exact visible asset role, crop, and source intent; do not substitute thumbnails, portraits, banners, or
  full-body images for each other
- Inspect directly linked detail pages before declaring requested fields unavailable; represent genuinely unavailable
  source data as unavailable or disabled instead of inventing values
- Verify app/game controls, keybindings, and menu paths from relevant code or docs before giving exact instructions.
- Validate the exact reported record, field, asset identity, and UI state; string presence, successful builds, and
  deploy status are insufficient.
- Verify final served media by MIME type, dimensions, or inspected render, not only filename or URL.
