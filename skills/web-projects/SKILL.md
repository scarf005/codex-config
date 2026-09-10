---
name: web-projects
description: Frontend implementation and validation rules. Use when creating or modifying web UI, frontend apps, Preact, Vite apps, or browser-visible behavior.
---

# Web Projects

- Bootstrap web projects from `https://github.com/scarf005/preset`.
- Keep UI minimal: no extra copy, branding, badges, links, gradients, animation, or border radius unless requested or
  required.
- Match existing component styling before accepting native/browser or design-system defaults that change the visual
  language. For creating or extending visual assets, apply `source-backed-changes` before choosing or generating them.
- For screenshot or recording complaints, identify the exact visible view, component, toolbar, state, and viewport
  before editing
- For CSS state styling, do not reuse generic state classes across unrelated components; check structural selectors for
  collisions
- Default to white theme; support `prefers-color-scheme`.
- Preact: prefer Signals/deepsignal; avoid hooks unless architecture/library boundaries require them.
- Preact event handlers: use `event.currentTarget.value` directly; do not cast typed targets like
  `(event.currentTarget as HTMLInputElement).value`.
- TypeScript/JavaScript in web projects: prefer `const` arrow functions for helpers, components, callbacks, and exported
  utilities. Use `function` when the syntax or API requires it (e.g., overload declarations, generators, methods).
- Use Vite `base: "./"` only when the hosting path requires relative assets; otherwise follow the existing project
  convention.
- Test web projects and browser-visible changes with `playwright-cli`; use the repo's actual dev server/routes for
  screenshots when available, create a temporary real page fixture when needed, and capture the full page unless the
  user explicitly asks for a cropped element.
- When a PR needs screenshot evidence, upload the images to a gist and link the raw gist URLs in the PR body instead
  of committing the image files.
- Validate the exact reported interaction at the reported viewport plus adjacent common viewports and inspect captured
  results before claiming overflow, scrollbar, alignment, or sizing fixes
- For visual changes, inspect the rendered result alongside the original/reference at the intended viewing scale.
  Check the task's visual acceptance criteria separately from interaction tests; a captured screenshot or successful
  Playwright run alone is not evidence of visual quality. Correct visible mismatches before claiming completion.
- For procedural layouts, define spatial relationships, functional regions, routes, spacing, and density before
  scattering individual objects. Preserve required relationships when placing groups; inspect several reproducible
  seeds both as an overview and at normal viewing scale. Collision-free placement alone does not establish coherence.
- Do not claim a live URL fixed until the commit is pushed and deployment is verified; label local validation explicitly
- For web references, include direct links and check `/llms.txt` when relevant.
- When changing Deno tests, read only the relevant official documentation for the testing, snapshot, or fast-check
  feature involved.
