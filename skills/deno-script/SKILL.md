---
name: deno-script
description: Create or modify Deno automation scripts, including new standalone automation when no existing runtime applies.
---

# Deno Scripts

- For existing repositories, use the existing runtime and style; for new standalone automation, use TypeScript with
  Deno unless the user requests another runtime
- Use the repository formatter; for Deno projects run `deno fmt`. When creating a new Deno config, set
  `semiColons: false`; preserve the existing semicolon configuration in existing projects
- Run scripts with the narrowest required `--allow-*` permissions; do not use `-A` unless the task requires unrestricted access
- Prefer standard-library or JSR imports and declare them in the repository Deno config when reused
- Add focused Deno tests for nontrivial parsing, generation, or calculation logic
- Validate with `deno fmt --check`, `deno check`, and the relevant `deno test` command
